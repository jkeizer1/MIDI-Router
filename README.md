# MIDI-Router

![image](https://user-images.githubusercontent.com/17629970/133940938-1160de31-6f55-4634-9de7-f984f7d90cb2.png)

A MIDI port/channel router that runs in a web browser (Web MIDI). Bookmark several maps and switch between them in a click instead of fussing in a DAW. Written for a **MIDIPLUS 8x8 USB MIDI box**, which has no routing of its own. One HTML file, no libraries.

## Setup
- Use Chrome (tested). Other browsers untested.
- Web MIDI needs a secure page: open the file straight from disk (`file://`), from `localhost`, or from an `https://` server.
- Allow MIDI (including SysEx) when the browser asks.

## Using it
**Map.** One row per output port. For each row choose:
- *Input Port* — the input that feeds this output (one input can feed many outputs).
- *Input Channel Filter* — `none` (row off), `any`, or one channel.
- *Output Channel* — channel messages are re-sent on this channel; `none` blocks channel messages.
- *Humane Name* — your label, e.g. "Volca Keys".

Two devices with the same name (e.g. two Korg SQ-1s) show as `SQ-1 (1)` and `SQ-1 (2)`, numbered in the order the browser lists them. If one is unplugged, the other becomes `(1)`, so re-check those rows after re-plugging.

**Save.** *Save map as link* copies a link to the clipboard and puts it in the address bar — press Ctrl+D to bookmark. Open the bookmark to get that map back.

**v0.07 bookmarks** still load: edit the bookmark and change `midirouter7_public.html` to `midirouter.html`; keep everything from `?Parm1=` on.

**Router on/off.** Stopping the router sends All Sound Off + All Notes Off (CC120, CC123, all 16 channels) to every output in use.

**PANIC** sends the same to every connected output, router on or off.

**System msgs** — BLOCKED by default. PASSING forwards clock, start/stop/continue, song position and SysEx from each input to all its outputs (unchanged; output channel doesn't apply). Use BLOCKED to stop sequencers such as Volcas starting by themselves.

**Plugging and unplugging** is picked up live. A row whose device is missing shows "not plugged in" and starts routing again when it comes back. If an input is unplugged while running, its outputs get notes-off.

**Problems** appear in a red box at the top with what to do and the technical detail. Nothing fails silently.

## Notes
- Hobby project; not hardened for security. The saved map lives in the link's `Parm1` parameter and is validated on load.
- One input per output: two inputs can't yet be merged into one output. Multi-input routes are planned.
- No note tracking: a changed row or stopped router relies on CC120/CC123, which nearly all synths honour.

## MIDI background
- https://www.cs.cmu.edu/~music/cmsip/readings/MIDI%20tutorial%20for%20programmers.html
- https://www.w3.org/TR/webmidi/

## Changes in v1.0 (from v0.07)
- Fixed: Chrome "form field should have an id or name" issues (one per table control).
- Fixed: "all notes off" on stop never worked — it re-sent the last incoming message instead, and only when the next message arrived. Now sends real CC120/CC123 at once.
- Fixed: each row had a stray 7th cell; malformed `</>` tags; invalid CSS.
- Fixed: a damaged link or a send to an unplugged device failed silently (console only).
- Fixed: Humane Names containing `?` broke loading.
- Fixed: "Real Time Start" toggle actually gated all system messages; now labelled as such.
- Fixed: Save link no longer needs the host name edited into the script.
- New: PANIC button, hot-plug, duplicate-name ports, notes-off on map change while running, strict mode with type-checked JSDoc, faster per-message routing (no table reads per message).
