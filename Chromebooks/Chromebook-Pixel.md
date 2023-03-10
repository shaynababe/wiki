The Pixel is fully supported by crouton with only a few minor caveats you need to keep in mind.

When issuing the command to build your chroot, you will want to add `-t touch` to your command. If you are adding a desktop environment target such as Xfce or Unity, you can combine it with touch via something like `-t touch,xfce`.

Be aware that until your favorite window manager supports high-DPI displays, results may vary.  Users are having good luck using Xfce with a few minor tweaks:

#### Option #1 - Use X11's `-dpi` flag:

 1. `sudo enter-chroot`
 2. Copy /etc/crouton/xserverrc-local.example to /etc/crouton/xserverrc-local, uncomment the XARGS line and modify the DPI value.


#### Option #2 change dpi settings in Xfce:
Support for dpi changes are quite spotty... but this way you get to keep text sharp.

 1. In Xfce, right-click the desktop and choose Desktop Settings.
 2. In Icons tab, increase the icon size. Recommended: 72.
 3. Close the window.
 4. Right-click the desktop and choose Applications > Settings > Appearance.
 5. In the Icons tab, select Humanity, Humanity-Dark, or Tango. GNOME icons are not resolution independent.
 6. In Fonts tab, enable Custom DPI and set it to 150, or adjust to your liking.
 7. Close window.
 8. Right-click the desktop and choose Applications > Settings > Panel.
 9. Use the slider to increase the size of the panel.  Recommended for ease of touch: 72.
 10. Close the window.
 11. Get a HiDPI Window Manager theme (https://github.com/mntmn/xfce4-hidpi-theme)
 12. Extract in /usr/share/themes/
 13. Enable it: Applications > Settings > Window Manager .
 14. Switch the GTK style: Applications > Settings > Appearance > Style .

This should give you an interface normal humans are capable of working with. If you have any more tips, please share them!

#### Option #3 change resolution
Trades the extreme definition for a more standard resolution that works perfectly with any window manager.  You can use the included `setres` script to change your resolution to something custom.  The script does not do sanity-checking, so you may end up with a blank screen and have to kill the chroot from Chromium OS if you give a strange resolution. Some examples:
```
setres 1920 1280
setres 1680 1120
setres 1440 960
setres 1280 850
```
You can make one of these resolutions persistent by following these steps:

1. Generate a modeline:
```
cvt 1920 1280 60
```
It will output something like:
```
Modeline "1920x1280_60.00"  206.25  1920 2056 2256 2592  1280 1283 1293 1327 -hsync +vsync
```

2. Create a 10-monitor.conf file:
```
sudo vim /usr/share/X11/xorg.conf.d/10-monitor.conf
```

You should get something like this:
```
Section "Monitor"
  Identifier "Monitor0"
  Modeline "1920x1280_60.00"  206.25  1920 2056 2256 2592  1280 1283 1293 1327 -hsync +vsync
EndSection
Section "Screen"
  Identifier "Screen0"
  Device "eDP1"
  Monitor "Monitor0"
  DefaultDepth 24
  SubSection "Display"
    Depth 24
    Modes "1920x1280_60.00" "1280x960"
  EndSubSection
EndSection
```

If you want a different screen resolution, replace the Modeline line with the one generated by `cvt` for you, and add the mode to the Modes line.

#### Option #4 set screen size on Xorg so that dpi goes up to 240

Add a file /etc/X11/xorg.conf.d/90-monitor.conf to explicitly set the screen size, i.e. containing

```
Section "Monitor"
    Identifier             "<default monitor>"
    DisplaySize            270 180    # In millimeters
EndSection
```

These numbers are actually taken from the Xorg log file in /var/log -- no clear to me why the correct numbers don't end up being used.  Not all apps handle this correctly

#### Gnome and Unity Troubleshooting
Sometimes when running in {gnome,unity}-xiwi, your session may get stuck with a dialog box complaining about the followings
```
   Could not apply the stored configuration for monitors
   none of the selected modes were compatible with the possible modes:
   Trying modes for CRTC 63
   CRTC 63: trying mode 1024x768@85Hz with output at 2200x1508@60Hz (pass 0)
   CRTC 63: trying mode 2048x1536@85Hz with output at 2200x1508@60Hz (pass 0)
   CRTC 63: trying mode 2048x1536@75Hz with output at 2200x1508@60Hz (pass 0)
   CRTC 63: trying mode 2048x1536@60Hz with output at 2200x1508@60Hz (pass 0)
   CRTC 63: trying mode 1920x1440@85Hz with output at 2200x1508@60Hz (pass 0)
   CRTC 63: trying mode 1920x1440@75Hz with output at 2200x1508@60Hz (pass 0)
   ... 
```
The solution (found here http://ubuntuforums.org/showthread.php?t=1852813) is to simply remove your ~/.config/monitors.xml.

```
   sudo enter-chroot
   mv .config/monitors.xml .config/monitors.xml.bkp
```

Then restart your session.

### HiDPI In XiWi Window

If you start one single application in XiWi window like this:

    sudo startxiwi -n chroot_name app_name

To support HiDPI in your application window, you need to:

1. Check HiDPI in your "Crouton Integration" extension. (By doing this alone, the font will be super small).

2. Edit your chroot:
```
    sudo enter-chroot
    sudo vi /etc/crouton/xserverrc-xiwi
    add "-dpi 135x135" at the end of XARGS line.
```

Start your application again with startxiwi. Notice that the setting will be reset every time you update your chroot. Backup the file and re-do it after every update...

NB. On my machine, this generated an X server error with unknown option "-dpi 135x135". According to the options list, this should be an integer value but even so I couldn't get it to work. Easiest is to set the resolution and font sizes in Xcfe appearance menus, as described above, using the graphical tool to get the display resolution correct. Then any application should display fine _without_ needing to have the HiDPI option checked in the extension.  

## Installing the latest version of Blender
The 2015 Pixel Chromebook is a beast even by today's (2018) standards. At the time, everyone wondered what the point of all ridiculous amount of RAM and processor speed was. Now we know.  :-)

By default, the Blender that ships with an LTS version of ubuntu (xenial at present) is several versions out of date, and all the cool stuff that you so desperately need isn't there, right?  ;-)

So, you first need to enable PPA to get repo with the latest blender:

`sudo apt-get update && sudo apt-get install python-software-properties` 

You may also need to install:

`sudo apt-get install software-properties-common`

Now you can add Thomas' personal repo for blender:

`sudo add-apt-repository ppa:thomas-schiex/blender`

Finally you need to update and install blender:

`sudo apt update && sudo apt install blender`

And you should have the latest version:

`(xenial)xxxx@localhost:~$ blender -v`

`Blender 2.80 (sub 45)`

Now, get ~on with some work~ modelling and rendering!

