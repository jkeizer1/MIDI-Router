# MIDI-Router
MIDI Router -
  ![image](https://user-images.githubusercontent.com/17629970/133940938-1160de31-6f55-4634-9de7-f984f7d90cb2.png)
<BR>
SETUP<BR>
-- host this on some webserver<BR>
-- enable SSL and provide a certificate; WEBMIDI requires SSL.  It still works if you bypass the broswer warnings (not recommended)<BR>
<BR>
USAGE<BR>
-- Example use with MIDIPLUS 8x8 MIDI box<BR>
<BR>
A) SETTING UP THE MAP<BR>
-- per the image, 8 input ports, 8 output ports<BR>
-- assign each input port to an output port<BR>
-- assign Input Channel [any, none, all] to filter channel mesages that will be sent to the Output Channel<BR>
-- assign the ouput channel<BR>
-- Give each map row a Humane Name so that you do not have to remember which device is using what port/channel combination<BR>
<BR>
B) SAVING THE MAP<BR>
-- press the "SAVE URI to clipboard" button (this simply creates a URI string with the parameters of the map table set)<BR>
-- paste this into your browser, and save a bookmark<BR>
<BR>
C) USING A PREVIOUS MAP<BR>
-- click the bookmar; the program will load the table from the URI parameters<BR>
<BR>
D) REFRESHING A MAP THAT YOU HAVE CHANGED<BR>
-- click the book mark again or clice the "Load template from URI" button<BR>
<BR>
E) STARTING THE ROUTER/STOPPING THE ROUTER<BR>
-- the router is either on or off, use the "Router Status" button to start/stop the router<BR>
<BR>
F) REAL TIME START MESSAGES<BR>
-- this feature disables/enables sending of the MIDI Real Time Start Messages<BR>
-- if enabled, your devices may start their own sequencers; this button allows you to disable this behaviour<BR>
-- for example, prevent a set of Volcas from always starting their sequencers<BR>
<BR>
G) CODE NOTES<BR>
-- this is a home project, and has not been hardened for Security at all<BR> 
-- the javascript is probably not up to best practices as it was only my second or third'ish script, could easily be improved and cleaned up in several ways; maybe later :)<BR>
-- this is my first WebMIDI project; use at own risk. The script has worked for my purposes; your milage may vary<BR>
<BR>
H) SOME MIDI BACKGROUND<BR>	
https://www.cs.cmu.edu/~music/cmsip/readings/MIDI%20tutorial%20for%20programmers.html <BR>
https://www.songstuff.com/recording/article/midi_message_format/ <BR>
I) SECURITY<BR>
-- Per (G), this is a home project.  Other than enabling SSL, I have NOT considered security aspects, for example effects of people hacking the URI string<BR>
-- Decide for yourself if this seems insecure and improve it if you need to; I make no representation that he code / solution is secure in any way; use at your own risk<BR>
<BR>
J) LIVE PERFORMANCES<BR>
-- I use this only to create music for hobby purposes; so far it has worked really well.  But I am not certain I would use this in front of an audience live; probably the biggest issue might be stuck notes; I have not encountered this issue too frequently and have not included a all notes off button.   <BR>
END<BR>
