Based upon: https://www.dahetral.com/public-download/alps-psmouse-dlkm-for-3-2-and-3-5/view

History
-------

1.3.1: Updated to support kernel >= 4.15 (using timer_setup(...) instead
        of setup_timer(...)) ~grufwub
        
1.3: Rebase to Kevin Cernekee's Alps mods using the 3.7 kernel.  Added
     init code for the Dell N5110 and segment init code for the 
     Fujitsu a512, which appear to have the same signature but different
     init sequences.
     
1.2: Add ACPI tests for Alps touchpad.

1.1: Patch format with bug fix from Malte Skoruppa for E6530.

1.0: Cleanup and put in Linux canonical patch format, remove all debug.

0.5: elevate persistent syslog to psmouse_info for production/release capture
      of alps signature and command response - both of which determine control 
      protocol.
      
0.4: integrate multi-touch support from florin9doi
     default alps_debug=0, can be enabled through sysfs
     code cleanup
     
0.3: clean up a potential null pointer exception
     added signature for Dell I17R 7720 based on user confirmaton to V6 protocol
     disable multi-touch for V6 protocol
     include alps.sh helper scripts
     
0.2: add logic to isolate N5110 changes from previously supported devices
     expand diagnostics to capture touchpad protocol
     add kernel parameter to enable/disable runtime packet debug
     enabled edge scrolling for N5110
     add more shell scripts to control/debug device
     
0.1: hacked in rudimentary support for the Dell IR15 N5110.  Nothing else 
     should work - probably causing a kernel panic.


Background
----------
This README documents my work to reverse engineer the Alps touchpad on a
Dell N5110 for Linux.  When I first purchased the laptop and installed
Ubuntu 12.04, I noticed that the touchpad behaved erratically.  It would
randomly skip and home-to-cursor. A brief search showed that this
behavior has been a long-standing and well publicized issued.  To date
several other Alps touchpads have been reverse engineered but no one had
addressed this particular model.  

Dell has a driver for the touchpad for Vista and Win7.  I repeatedly
contacted Dell support and Alps; the only response I received was "We
don't support Linux"; odd considering Dell ships Ubuntu.

I developed a simple serio_raw program to peek/poke at the touchpad and
found it had characteristics inconsistent with other Alps touchpads.  I
used Seth Forshee's procedure to dump the driver interface using qemu and
vista as the guest OS [1].  This require me to patch the qemu
acpi-dsdt.dsl file to recognize the Alps hardware in order to install
the Dell/Alps Vista driver [2]

Based on the qemu log, I started hacking up a psmouse DLKM from the
linux-3.2.0 source tree.  First I added in Ben Gamari's ALPS_PROTO_V5
fix for the Alps Touchpad on his Dell E6230 [3] and tried it but the
characteristics were too different.  In fact even the enter-command-mode
generic response is different.  I then took the vista log and
created an ALPS_PROTO_V6 hw init by pasting the init sequence from the
qemu log into alps_hw_init_v6.  That got me part of the way, the device
was no longer being handled as a stock ps/2 mouse but now as an ALPS
DualPoint Touchpad.  However, the 6-byte message format was undocumented
[4]. As far as I can tell, it is:

 byte 0:    1    1    0    0    1    0    0    0
 byte 1:    0   x6   x5   x4   x3   x2   x1   x0
 byte 2:    0   y6   y5   y4   y3   y2   y1   y0
 byte 3:    0    ?    ?    ?    1    m    r    l
 byte 4:   y10  y9   y8   y7  x10   x9   x8   x7
 byte 5:    0   z6   z5   z4   z3   z2   z1   z0

So I created alps_process_touchpad_packet_v6 to parse the touchpad
packets.

The ALPS touchpad is working on my N5110.  I can control the touchpad
somewhat via xinput and the System Settings->Touchpad widget.  I think
it can still be tightened up but, at least, it's usable now.

Dave Turvene

[1] http://swapspace.forshee.me/2011/11/touchpad-protocol-reverse-engineering.html

[2] http://www.spinics.net/lists/linux-input/msg21948.html

[3] http://www.spinics.net/lists/linux-input/msg21993.html

[4] file:///usr/src/linux-source-3.2.0/linux-3.2.0/Documentation/input/alps.txt
