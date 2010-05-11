What
====

This script detects when you plug your external monitor on your laptop running nvidia proprietary driver, and automatically swiches to and from it.

I tested it only on a MacBook pro 5,3 and Ubuntu Lucid 10.4, so your mileage may vary.

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
