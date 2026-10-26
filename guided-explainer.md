# MIDI Router v1.0 — Guided Explainer

For the next maintainer of `midirouter.html`. It assumes you read C, C++ or Java comfortably and have touched JavaScript, but haven't lived in it. It explains the program in reading order and maps the JavaScript idioms it uses to things you already know.

Read this alongside the source. The script is split by banner comments (`// ---------- Types ----------`, and so on); this guide follows the same order.

---

## 1. The whole thing in one paragraph

One HTML file: some CSS, a static page skeleton (buttons, an empty table body, a problems box), and one `<script>`. On load, `start()` wires the buttons and asks the browser for MIDI access. When access is granted, the script lists the connected ports, builds one table row per output port, and compiles the rows into a **routing table**: a map from input port id to the list of outputs it feeds. From then on the browser calls a handler for every incoming MIDI message; the handler looks up the input's routes, remaps the channel, and sends the bytes on. Plugging or unplugging a device rebuilds the port list, the table and the routing table. Saving encodes the rows as JSON in the page's URL.

If you have done EAI work, this is a tiny message broker: **configuration** (what the musician chose) is kept apart from a **compiled routing table** (what the message path reads), and the message path never touches the UI.

---

## 2. The runtime model: one thread, run to completion

This matters more than any syntax.

- A web page's JavaScript runs on **a single thread**. The browser keeps an event queue; each event (a click, an incoming MIDI message, a timer) runs its handler **to completion** before the next one starts. There is no preemption and no shared-memory concurrency, so the code has no locks and needs none.
- Closest analogies: Unity's main thread, where every `Update()` and callback runs on one thread; or a Lua host calling your callbacks one at a time.
- Consequence 1: a slow handler delays everything, including MIDI. That is why the per-message path (`onInputMessage`) does a map lookup and a short loop and nothing else, and why the message counter is redrawn at most once per screen frame (`scheduleCounterRedraw` uses `requestAnimationFrame`) instead of on every message.
- Consequence 2: when a function replaces a global (for example `routesByInputId = next`), no MIDI handler can observe a half-built table. The next message sees the new table.
- Web MIDI delivers **one complete message per event**. Running status is already expanded, and SysEx arrives whole. `ev.data[0]` is always the status byte.

---

## 3. JavaScript features used, mapped to what you know

| In this file | What it is | Nearest familiar thing |
|---|---|---|
| `"use strict";` | Turns silent mistakes into errors (assigning to an undeclared variable, writing a read-only property). | Compiler warnings promoted to errors. v0.07's notes-off bug (assigning to a read-only `message.data`) would have thrown here instead of failing silently. |
| `// @ts-check` plus `/** @type {X} */`, `/** @param {X} name */` | Type annotations written as **comments**, checked by the TypeScript compiler as an external lint step. The browser ignores them; **no runtime enforcement**. | Java types, but checked only when you run the checker (see §10). |
| `/** @typedef {{kind:"none"} \| {kind:"channel", channel:number}} OutputChannel */` | A **tagged union**: an object whose `kind` field says which shape it is. After `if (c.kind === "channel")` the checker knows `c.channel` exists. | Rust `enum`, Scala sealed trait plus case classes, a C tagged union. |
| `const` / `let` | Block-scoped variables. `const` means the *binding* can't be reassigned; the object it points to can still be mutated. | `final` in Java (reference final, object mutable). Never use `var` (function-scoped, hoisted). |
| `class X { constructor(a) { this.a = a; } }` | A real class. Fields are created by assignment in the constructor. No overloading; one constructor. | A Java class with only public fields and one constructor. |
| `(ev) => { ... }` | Arrow function (lambda). Captures surrounding variables **by reference**. | Java lambda, or a Lua closure with upvalues. |
| `Map`, `Set` | Hash map and hash set that **iterate in insertion order**. `map.get(k)` returns `undefined` when the key is absent. | `LinkedHashMap` / `LinkedHashSet`. |
| `m.forEach((value, key) => …)` | Note the order: **value first**, key second. | The reverse of what a Java `Map.Entry` habit suggests. v0.07 named these backwards. |
| `===`, `!==` | Equality without type coercion. Always use these, never `==`. | `==` on primitives in Java. |
| `a ?? b` | "`a` unless it is `null` or `undefined`, else `b`." | Kotlin's `?:`, `Optional.orElse`. |
| `null` vs `undefined` | Two "nothing" values. The browser APIs return `null` (e.g. a port's `name` may be null); `Map.get` returns `undefined`. The code checks for whichever the API documents. | — |
| `x instanceof HTMLButtonElement` | Runtime class test. Also narrows the type for the checker. | Java `instanceof` before a cast. |
| `Promise`, `.then(ok, fail)`, `async` / `await` | A value that arrives later. `await` pauses *this function* (not the thread) until it resolves; other events keep running meanwhile. | `CompletableFuture`; a C# or Scala `Future`. |
| `Uint8Array` | A typed byte array (MIDI data arrives as one). | `byte[]`. |
| `Array.from(x)` | Copy any array-like into a plain array. | `Arrays.asList(...)` into a new `ArrayList`. |

**Hoisting and order.** Named `function` declarations are usable anywhere in the script (they are "hoisted"). Classes and `let`/`const` are not usable before their line runs. The script calls `start()` on its very last line, so everything is defined by then.

---

## 4. The page and the DOM, briefly

The HTML at the top is static, except for `<tbody id="routeTableBody">`, which the script fills in. The script finds elements by id through three small helpers, `el`, `button` and `textInput`. Each one **throws** if the id is missing or the element is the wrong kind, so a mismatch between HTML and script fails loudly instead of returning `null`.

The DOM is the page as a tree of objects. `document.createElement("td")` makes a node, `appendChild` attaches it, `textContent = "…"` sets its text safely (no HTML parsing, so port names can't inject markup), and `addEventListener("change", fn)` registers a callback. That covers everything this program does with the page.

---

## 5. The data model (section "Types")

All types are purpose-named classes, one per job:

| Type | Lifetime | Role |
|---|---|---|
| `PortRef` | value | Identifies a port as a **saved link** does: `name` plus `ordinal`, which says which one among ports of the same name (1, 2, …). `key()` turns it into the string used as a `<select>` option value (`"2:SQ-1"`). |
| `ListedInput` / `ListedOutput` | until the next re-list | A **currently connected** port: the browser's `MIDIInput`/`MIDIOutput` object, its `PortRef`, and the label shown in the UI. |
| `RowSetting` | whole session | **Configuration** for one table row, i.e. one output: which input feeds it, the input channel filter, the output channel, the humane name. It refers to ports by `PortRef`, not by live object, so it survives unplugging. |
| `Route` | until the next rebuild | **Compiled** routing entry: a live `MIDIOutput` plus filter and channel. Only the message path reads these. |
| `SavedRow` | transient | Exact shape of one JSON row in a saved link. Field names are kept from v0.07. |
| `ErrorRepeat` | session | Throttle state for one repeating error. |

Why `PortRef` and not the browser's port id? Saved links must work on another day, after re-plugging, when ids may differ. Name plus ordinal is the most stable identity the browser offers. Its weakness: with two identical devices, unplugging the first renumbers the second.

**Module state** (section "State") is a handful of `let` globals: the MIDI access object, the run flags, `rowSettings`, the two listed-port arrays, `routesByInputId`, and so on. Treat them as the program's single model. They are replaced or updated only by the functions named below.

---

## 6. The data flow, end to end

```
browser ports ──listPorts()──► listedInputs / listedOutputs
                                       │
saved link ──settingsFromSavedJson()──►│
                                       ▼
                     reconcileRowSettings()  ──►  rowSettings  ◄── UI edits (closures in renderTable)
                                       │
                         ┌─────────────┴──────────────┐
                         ▼                            ▼
                   renderTable()               rebuildRoutes()
                   (DOM rows)                         │
                                                      ▼
                                              routesByInputId   ◄── read by onInputMessage() per MIDI message
```

Whenever ports change or settings change, the sequence is: re-list ports, reconcile settings, render the table, rebuild routes. `onMidiReady`, `onPortStateChange` and `onReload` each run that sequence. `onRowRoutingChanged` runs only the last step, because the ports didn't change.

---

## 7. Walkthroughs

### 7.1 Start-up: `start()` → `onMidiReady()`

1. `start()` attaches click handlers. Each handler is wrapped by `guarded(...)` (§8).
2. It checks `window.isSecureContext`, because Web MIDI is only offered on `https://`, `localhost` or `file://` pages, and checks that `navigator.requestMIDIAccess` exists.
3. `requestMIDIAccess({ sysex: true })` returns a Promise. The browser may show a permission prompt. On success `onMidiReady(access)` runs; on refusal the reason goes to the Problems box.
4. `onMidiReady` runs the §6 sequence once, attaches a handler to every input (`attachInputHandlers`), subscribes to plug/unplug (`access.onstatechange`), and enables the buttons.

### 7.2 A MIDI message: `onInputMessage(input, ev)`

This is the hot path; read it first if you read nothing else.

1. If the router is off, return.
2. Check that the event really is a `MIDIMessageEvent` with data (a defensive check; failures are reported, not ignored).
3. `routesByInputId.get(input.id)`. No routes for this input means nothing to do.
4. Split on the status byte:
   - **0x80–0xEF, channel messages.** `inChannel = status & 0x0F`, `kind = status & 0xF0`. For each route: skip it if the input filter doesn't pass this channel or the output channel is `none`; otherwise copy the bytes, set `out[0] = kind | outputChannel`, and `sendTo`. Data bytes are untouched, so note, CC, program change and pitch bend all work the same way.
   - **0xF0–0xFF, system messages** (clock, start/stop, SysEx). Forwarded unchanged to every route of this input, only when "System msgs" is PASSING.
   - **Below 0x80** can't happen with Web MIDI; if it ever does, it's reported.
5. `sendTo` wraps `output.send()` in try/catch. Sending to a vanished port throws, and the error is shown, throttled to once per 2 s per output so a stream of notes can't flood the page.

`output.send(bytes)` with no timestamp means "send now". Web MIDI also accepts a timestamp argument for scheduled sends; this program doesn't use it.

### 7.3 Editing a row: the closures in `renderTable()`

`renderTable()` empties the table body and rebuilds every row from `rowSettings`. For each row it creates the three `<select>`s and the text box, and registers a `change` listener on each. Each listener is an arrow function that **captures that row's `RowSetting` object (`rs`)**. When you pick a new value, the listener writes straight into `rs` and calls `onRowRoutingChanged(rs)`.

`onRowRoutingChanged` sends notes-off to that output if the router is running (held notes would otherwise never get their note-off through the changed route), then `rebuildRoutes()`.

If you're used to MVC with explicit binding, this is binding by closure: the DOM control and its model object are linked by the captured variable, not by an id lookup.

Each control gets a unique `id` and `name` (`inPort3`, `outChannel3`, …) and an `aria-label`. Chrome lists any control without an id or name as an issue in DevTools; this is what produced the "80 errors" in v0.07.

### 7.4 Plug and unplug: `onPortStateChange()`

The browser fires `statechange` on the MIDI access object for real plug/unplug events **and also** whenever a port is merely opened or closed, which happens implicitly on first send. To tell them apart, the code keeps `portSignature`: a sorted string of all connected port ids. If the signature hasn't changed, it's only an open/close and the handler returns.

On a real change, it:
1. finds outputs that were fed by an input that has just disappeared, and remembers them;
2. runs the §6 sequence;
3. if the router is running, sends notes-off to those remembered outputs;
4. shows a one-line notice ("Input "X" unplugged.").

Disconnected ports stay in the browser's port maps with `state === "disconnected"`; `listPorts` keeps only `"connected"` ones. `reconcileRowSettings` keeps a row for an unplugged output only if it has been configured, so a device you never set up doesn't leave a ghost row.

### 7.5 Save and load: section "Saved links"

- **Format:** `?Parm1=` + URL-encoded JSON `{ version, uriParms: [SavedRow…] }`. The field names are v0.07's, plus two optional ordinals (`input_port_index`, `output_port_index`).
- **Save (`onSave`)** is `async` because the clipboard API returns a Promise. It writes the link into a visible text box, puts it in the address bar with `history.replaceState` (so Ctrl+D bookmarks it), and copies it. Each step reports its own failure.
- **Load (`settingsFromSavedJson`)** treats the JSON as untrusted. It parses into `unknown`, checks every field's type and range (`requireString`, `parseChannel`, `optionalOrdinal`), and throws a message naming the bad row and field. The caller shows that message and keeps the current map. The one default it applies: a missing ordinal means 1, which is exactly how v0.07 links behaved (first port of that name).

### 7.6 Notes off: `sendPanic()`

Builds one 96-byte packet: for each of 16 channels, CC120 (All Sound Off) then CC123 (All Notes Off). `send()` accepts several complete messages in one call. It is used when the router stops (routed outputs), on the PANIC button (all connected outputs), on a row edit while running (that output), on map reload, and when an input is unplugged while running. There is no per-note tracking.

---

## 8. Error handling: section "Error reporting"

The rule is **nothing fails silently**. Three layers:

1. **`guarded(context, fn)`** wraps every event handler (buttons, table controls, MIDI input, plug/unplug). It is a function that returns a function, the decorator pattern done with closures. Any exception is caught and shown with the handler's context string. `guardedAsync` does the same for Promise-returning handlers.
2. **Specific try/catch** where a failure has a specific remedy (send, clipboard, address bar, saved link), each with a plain instruction for the musician.
3. **Global backstops**: `window` listeners for `error` and `unhandledrejection` catch anything that escapes, so even a bug in the wrappers shows up on the page.

`showError` writes a headline (what to do, in plain words), the context and the full stack into the Problems box, and also logs to the DevTools console. `showRepeatingError` throttles by key, for errors that can repeat at MIDI speed.

When you add code, either keep it inside an existing `guarded` handler or wrap the new handler. Don't write `catch (e) {}`.

---

## 9. Known limits (and where to change them)

- **One input per output.** The table is one row per output (`reconcileRowSettings`, `renderTable`). The compiled `routesByInputId` already allows several routes to the same output, so multi-input routes need table, save-format and notes-off changes, not message-path changes. If you merge inputs, watch two things: clock arriving from two sources, and the same note number from two inputs on one output channel.
- **Duplicate-name identity** depends on browser enumeration order (§5).
- **No note tracking**; relies on CC120/CC123.
- **System messages** are a single on/off for all inputs. Splitting clock from start/stop would happen in the `status >= 0xF0` branch of `onInputMessage`.

---

## 10. Checking your changes

There is no build step. Edit the file and reload the page.

**Type check** (needs Node.js; `npx` downloads TypeScript on first run, and nothing is added to the project):

```
sed -n '/<script>/,/<\/script>/p' midirouter.html | sed '1d;$d' > /tmp/router.js
npx -p typescript@5 tsc --checkJs --allowJs --strict --noEmit --target es2022 --lib dom,es2022 /tmp/router.js
```

No output means no type errors. The extraction step exists because `tsc` can't read a script embedded in HTML.

**Chrome DevTools** (F12): *Console* shows the `[MIDI Router]` error log; *Issues* should stay empty.

**Tests:** v1.0 was tested in headless Chromium against a mock Web MIDI implementation (fake ports, recorded sends), covering routing and remap, filters, system messages, duplicate names, plug/unplug, notes-off, save and load, damaged links and refused permission. Those scripts are not in this repository. On real hardware, at minimum check: routing through a few rows, channel remap, stop and PANIC silencing a held note, and unplugging and re-plugging a device mid-session.

---

## 11. Gotchas the type checker taught us

- TypeScript's built-in browser types declare `MIDIOutput.send()` as taking `number[]`, although browsers also accept a `Uint8Array`. The code converts with `Array.from(...)` to keep the checker strict.
- The same type definitions omit `get()` on `MIDIInputMap`, although browsers have it. The code iterates with `forEach` instead.
- `<select>.value = "x"` silently does nothing if no option has value `"x"`. `makeSelect` checks afterwards and throws.
