What
====

This script detects when you plug your external monitor on your laptop running nvidia proprietary driver, and automatically swiches to and from it.

I tested it only on a MacBook pro 5,3 and Ubuntu Lucid 10.4, so your mileage may vary.

Why
===

I just wanted to have the display working seamlessly as on macosx. I'm constantly switching from external monitor to pure laptop mode and I found it very frustrating
to use the nvidia-settings GUI tool: it required far too much clicks to get the job done.

The script started as a simple command-line wrapper around the nvidia-settings functionality.

I found out that the command-line interface provided by nvidia-settings was not powerful enough to set the "panning" while switching the mode.
I could switch to the external monitor and back to the laptop monitor, but once back it ended up with 1440x900 mode and 1680x1050 "panning": the desktop didn't resize
and you could pan through a larger area by moving with the mouse agains the screen edge.

How does it work
================

By skimming into the nvidia docs and nvidia-settings sources, I noticed that what nvidia-settings actually does is communicating to the nvidia driver
and settings nvidia specific "metamodes", which are configurations of resolution+screen. Those modes can be added dynamically or configured inside the xorg.conf file.

nvidia-settings then uses plain xrandr 1.1 (xrandr -s <size index> -r <refresh rate>) to swich to a given metamode configuration.

The thing is a little kludgy since nvidia maps a metamode configuration to a combination of "screen size" (computed as the bounding box of all the screens combined)
and an arbitrary "refresh" rate number, which is used to disambiguate when there is more than one metamode having the same total bounding box.

nv-control-dpy utility is a utility created to demonstrate the nvidia API and provides command line access to those low level features missing from the main tool.
I didn't want to waste time creating a C application interacting directly with the API, so I created a bash hack around that utility.

When both displays are available upon X server startup, the nvidia driver adds the necessary metamodes (from xorg.conf) and the script simply
detects the "size index" and the "refresh rate" by parsing the metamodes and xrandr output.

The tricky part is when the X servers starts up without the external monitor connected: even if you put the metamode conf option in xorg.conf, the metamode is not added.
The script thus detects when the external monitor is connected but not correctly initialized and adds the display and a suitable metamode (currently hardcoded to my preferred resolution :-( )

Since my personal need was to automate the whole thing, the script loops endlessly and detects when the external monitor gets (dis)connected and reacts by switching to the external monitor whenever it's available.

Requirements
============

 * nv-control-dpy executable. You have to compile it from nvidia-settings-1.0/samples (you can get the tarball from ftp://download.nvidia.com/XFree86/nvidia-settings/nvidia-settings-195.36.24.tar.gz) and put it somewhere in the PATH.
 * sudo apt-get install build-essential libxv-dev libxxf86vm-dev    and possibly other deps are required to build nvidia-settings
 * I had to delete the bundled "nvidia-settings-1.0/src/libXNVCtrl/libXNVCtrl.a" because it was pre-compiled for i386. It's correctly rebuilt by 'make'.
 
Usage
=====

I've put this line in xorg.conf:


	  
    Section "Screen"
	 ...
	 Option         "metamodes" "DFP-1: nvidia-auto-select +0+0, DFP-0: NULL; DFP-0: nvidia-auto-select +0+0, DFP-1: NULL"
	 ...

With this configuration I get primary display on external monitor by default if it's connected at boot, otherwise it falls back to the internal monitor.

My laptop doesn't set correctly the DPI when the external monitor is found at boot, because the X session starts without the internal monitor. I had to force it with:

    Section "Device"
	  ...
	  Option "DPI" "110x108"
	  ...
	 
Then you have to find a way to start the auto-display script when your session starts up.
	
Then I've created a gnome startup file which points to a startup.sh script which spawns the "auto-display" script in backround (among other things):

~/.config/autostart/custom.desktop:

    [Desktop Entry]
	Version=1.0
	Encoding=UTF-8
	Name=custom
	Type=Application
	Exec=/home/marko/bin/startup.sh
	Terminal=false
	Icon=gnome-do
	Comment=startup
	Categories=Utility;

~/bin/startup.sh:

    #!/bin/sh
	~/bin/auto-display &

You might also try to add it to gdm or whatever you are using.

ISSUES
======

I never tried it with anything other than my laptop. The resolutions and display numbers are hardcoded in the script.

The code is very alpha, it may leave you without a working X session, so take care of setting up a ssh server so you can get back in.

You might want to learn how to manually switch the screen back to the internal one. Usually this will work:

    xrandr -s 0
	
Anyway, take a look at xrandr output and choose the size index depending on the position of your laptop screen resolution in the list.

AUTHOR
======

Marko Mikulicic - marko.mikulicic@isti.cnr.it
ISTI - CNR
