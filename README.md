# MIDI-Router
MIDI Router -
  ![image](https://user-images.githubusercontent.com/17629970/133940938-1160de31-6f55-4634-9de7-f984f7d90cb2.png)
SETUP
-- host this on some webserver
-- enable SSL and provide a certificate; WEBMIDI requires SSL.  It still works if you bypass the broswer warnings (not recommended)
USAGE
-- Example use with MIDIPLUS 8x8 MIDI box
A) SETTING UP THE MAP
-- per the image, 8 input ports, 8 output ports
-- assign each input port to an output port
-- assign Input Channel [any, none, all] to filter channel mesages that will be sent to the Output Channel
-- assign the ouput channel
-- Give each map row a Humane Name so that you do not have to remember which device is using what port/channel combination
B) SAVING THE MAP
-- press the "SAVE URI to clipboard" button (this simply creates a URI string with the parameters of the map table set)
-- paste this into your browser, and save a bookmark
C) USING A PREVIOUS MAP
-- click the bookmar; the program will load the table from the URI parameters
D) REFRESHING A MAP THAT YOU HAVE CHANGED
-- click the book mark again or clice the "Load template from URI" button
E) STARTING THE ROUTER/STOPPING THE ROUTER
-- the router is either on or off, use the "Router Status" button to start/stop the router
F) REAL TIME START MESSAGES
-- this feature disables/enables sending of the MIDI Real Time Start Messages
-- if enabled, your devices may start their own sequencers; this button allows you to disable this behaviour
-- for example, prevent a set of Volcas from always starting their sequencers
G) CODE NOTES
-- this is a home project, and has not been hardened for Security at all 
-- the javascript is probably not up to best practices as it was only my second or third'is script, could easily be improved and cleaned up in several ways; maybe later :)
-- this is my first WebMIDI project; use at own risk. The script has worked for my purposes; your milage may vary
H) SOME MIDI BACKGROUND
// Info about MIDI Commands:	
// https://www.cs.cmu.edu/~music/cmsip/readings/MIDI%20tutorial%20for%20programmers.html
//https://www.songstuff.com/recording/article/midi_message_format/
I) SECURITY
-- Per (G), this is a home project.  Other than enabling SSL, I have NOT considered security aspects, for example effects of people hacking the URI string
-- Decide for yourself if this seems insecure and improve it if you need to; I make no representation that he code / solution is secure in any way; use at your own risk
J) LIVE PERFORMANCES
-- I use this only to create music for hobby purposes; so far it has worked really well.  But I am not certain I would use this in front of an audience live; probably the biggest issue might be stuck notes; I have not encountered this issue too frequently and have not included a all notes off button.   
