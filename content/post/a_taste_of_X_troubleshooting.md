+++
date = "2015-11-07"
title = "A Taste of X Server & nouveau troubleshooting"
description = "Adventures with Xorg trying to setup triple-head with nouveau"
keywords = ["X Server", "nouveau driver", "nvidia", "xorg.conf"]
menu = "How-To"
+++

 I am not responsible for your system
 or you mental state after reading this guide.
The purpose of this article is to help me understand how
stupid I am trying to setup my multi-head monitors.



## Physical Display confiruation:

This is what I would like to setup:

```
  ----------   ----------   ----------
 |          | |          | |          |
 | Monitor0 | | Monitor1 | | Monitor2 |
 |          | |          | |          |
  ----------   ----------   ----------
     | |          | |          | |
     | |          | |          | |
    /   \        /   \        /   \
    =====        =====        =====
     DVI          HDMI          VGA
      |            |             |
       ------      |     --------
             \     |   /
              \    |  /
              ===========
             | NVIDIA    |
             | GeForce   |
              ===========
```

Eventually, I didn't managed to set it up... Here's my
story...

## Try with dynamic detection

First all all, make sure there's no `/etc/X11/xorg.conf`
and if it exists, please remove it. As a side effect, X will try to 
configure your displays automatically.


Once you are logged into the Gnome session (fingers' crossed),
you are going to experience this:

```
  ----------   ----------   ----------
 |          | |          | |          |
 | NoSignal | |  Mirror  | |  Mirror  |
 |          | |          | |          |
  ----------   ----------   ----------
     | |          | |          | |
     | |          | |          | |
    /   \        /   \        /   \
    =====        =====        =====
     DEAD           |           |
                    |           |
                     -----------
                 Mirrored by default
```


Well, I suppose this is not the case that we want.
We need to explicitely tell X Server how to organise
our triple-head setup.



## Find out the Output names

While you're logged into Gnome, open a terminal 
and see the output names using `xrandr -q`:

    Screen 0: minimum 320 x 200, current 1152 x 864, maximum 8192 x 8192

        DVI-I-1 connected (normal left inverted right x axis y axis)
           1680x1050     59.88 +
           1280x1024     75.02    60.02  
           1152x864      75.00  
           1024x768      75.08    60.00  
           800x600       75.00    60.32  
           640x480       75.00    60.00  
           720x400       70.08  

        HDMI-1 connected 1152x864+0+0 (normal left inverted right x axis y axis) 473mm x 296mm
           1680x1050     59.95 +
           1280x1024     75.02    60.02  
           1152x864      75.00* 
           1024x768      75.08    60.00  
           800x600       75.00    60.32  
           640x480       75.00    60.00  
           720x400       70.08  

        VGA-1 connected 1152x864+0+0 (normal left inverted right x axis y axis) 527mm x 297mm
           1920x1080     60.00 +
           1600x900      59.98  
           1280x1024     75.02    60.02  
           1152x864      75.00* 
           1024x768      75.08    60.00  
           800x600       75.00    60.32  
           640x480       75.00    60.00  
           720x400       70.08


So, the corresponding names for our displays are:

    Monitor0 --> DVI-I-1
    Monitor1 --> HDMI-1
    Monitor2 --> VGA-1



## Create a very basic "xorg.conf"

Let's ask X Server to create a very basic "xorg.conf"
for us. To do that, shutdown any GUI by simply going into
Runlevel 3.

```~~> telinit 3```

Then, press `CTRL+ALT+F1` to go to the first tty and login back
to the system as root. Now, ask the X Server to serve you
the **xorg.conf** configuration file:

```~~> X -configure```

This will generate the "/root/xorg.conf.new" with some standard
configuration. Here it is:

    Section "ServerLayout"
        Identifier     "X.org Configured"
        Screen      0  "Screen0" 0 0
        InputDevice    "Mouse0" "CorePointer"
        InputDevice    "Keyboard0" "CoreKeyboard"
    EndSection

    Section "Files"
        ModulePath   "/usr/lib64/xorg/modules"
        FontPath     "/usr/share/fonts/misc:unscaled"
        FontPath     "/usr/share/fonts/Type1/"
        FontPath     "/usr/share/fonts/100dpi:unscaled"
        FontPath     "/usr/share/fonts/75dpi:unscaled"
        FontPath     "/usr/share/fonts/ghostscript/"
        FontPath     "/usr/share/fonts/cyrillic:unscaled"
        FontPath     "/usr/share/fonts/misc/sgi:unscaled"
        FontPath     "/usr/share/fonts/truetype/"
        FontPath     "built-ins"
    EndSection

    Section "Module"
        Load  "vnc"
        Load  "glx"
    EndSection

    Section "InputDevice"
        Identifier  "Keyboard0"
        Driver      "kbd"
    EndSection

    Section "InputDevice"
        Identifier  "Mouse0"
        Driver      "mouse"
        Option      "Protocol" "auto"
        Option      "Device" "/dev/input/mice"
        Option      "ZAxisMapping" "4 5 6 7"
    EndSection

    Section "Monitor"
        Identifier   "Monitor0"
        VendorName   "Monitor Vendor"
        ModelName    "Monitor Model"
    EndSection

    Section "Device"
        ### Available Driver options are:-
        ### Values: <i>: integer, <f>: float, <bool>: "True"/"False",
        ### <string>: "String", <freq>: "<f> Hz/kHz/MHz",
        ### <percent>: "<f>%"
        ### [arg]: arg optional
        #Option     "SWcursor"              # [<bool>]
        #Option     "HWcursor"              # [<bool>]
        #Option     "NoAccel"               # [<bool>]
        #Option     "ShadowFB"              # [<bool>]
        #Option     "VideoKey"              # <i>
        #Option     "WrappedFB"             # [<bool>]
        #Option     "GLXVBlank"             # [<bool>]
        #Option     "ZaphodHeads"           # <str>
        #Option     "PageFlip"              # [<bool>]
        #Option     "SwapLimit"             # <i>
        #Option     "AsyncUTSDFS"           # [<bool>]
        #Option     "AccelMethod"           # <str>
        Identifier  "Card0"
        Driver      "nouveau"
        BusID       "PCI:1:0:0"
    EndSection

    Section "Screen"
        Identifier "Screen0"
        Device     "Card0"
        Monitor    "Monitor0"
        SubSection "Display"
            Viewport   0 0
            Depth     1
        EndSubSection
        SubSection "Display"
            Viewport   0 0
            Depth     4
        EndSubSection
        SubSection "Display"
            Viewport   0 0
            Depth     8
        EndSubSection
        SubSection "Display"
            Viewport   0 0
            Depth     15
        EndSubSection
        SubSection "Display"
            Viewport   0 0
            Depth     16
        EndSubSection
        SubSection "Display"
            Viewport   0 0
            Depth     24
        EndSubSection
    EndSection



As you can see, it's quite generic. For this reason,
we are going to strip it down to a shorter version:

    Section "ServerLayout"
        Identifier     "TripleHead Setup with nouveau"
        Screen      0  "Screen0" 0 0
    EndSection

    Section "Monitor"
        Identifier   "Monitor0"
    EndSection

    Section "Device"
        Identifier  "Card0"
        Driver      "nouveau"
        BusID       "PCI:1:0:0"
    EndSection

    Section "Screen"
        Identifier "Screen0"
        Device     "Card0"
        Monitor    "Monitor0"
    EndSection



## What does it mean?

First of all we need a Monitor, which is equivalent to
our physical display. Normally, we would have 3 Monitor
sections, but hey: this is a very generic and basic, let's
stick to that for now on.

```
  ----------
 |          | The "Identifier" is just a name. You can put
 | Monitor0 | whatever you want. In this guide, I am sticking
 |          | with the "Monitor<N>" naming-convention.
  ----------
     | |
    /   \
    =====
```


Next we need to plug this monitor into our videocard and
make use of the videodriver:

```
   =================
  |                 |   The "Identifier" could be anything.
  | Card0           |   But, the important part here is the driver which is responsible
  |                 |   for the communication between the Card and the Linux kernel.
   =================
    ||||||||| |||           
    --------- ----         ###########
   | PCIe 01:00.0 | <--->  # nouveau # <---> KERNEL 
    --------- ----         ###########
```

Notice that the X found the open soure nvidia driver
used for 3D accelleration, called "nouveau". JYFI, you
should not use the old 2D `nv` driver, because is obsolete
and not maintained.

Just to be on the safe side, make sure that "nouveau" is loaded:

```~~> lsmod | grep drm```

        drm_kms_helper         65670  1 nouveau
        drm                   335594  6 ttm,drm_kms_helper,nouveau


and also, check that nouveau is the driver that userpace and kernel are using
for your videocard:

```~~> lspci -nnk | grep -A3 VGA```

        01:00.0 VGA compatible controller [0300]: NVIDIA Corporation GF108 [GeForce GT 440] [10de:0de0] (rev a1)
            Subsystem: ASUSTeK Computer Inc. Device [1043:83b7]
            Kernel driver in use: nouveau
            Kernel modules: nouveau


Notice that while the PCI BusID is: **"01:00.0"**
we change it a bit when we are putting this info
into `xorg.conf` like this: "PCI:1:0:0". This information is
optional if you have only one GPU installed (also, no internal graphics).
Otherwise, you have to specify it. For my system, I have Intel graphics
disabled from BIOS and I could have omitted that line from "xorg.conf"
but because there's always the danger to enable them sometime in the future,
I wouldn't like to end up with broken X. Because in that case, there will be
two PCI Buses:

  PCI:1:0:0 --> nVIDIA
  PCI:0:2:0 --> Intel Graphics

and if there's no reference into "xorg.conf" what is what, then it is going to
be a problem.

JFYI, the "0x10de" is the code for NVIDIA as a vendor
and the "0x0d30" is the code for the specific device (GeForce GT 440).
In case you need to validate these information, you can search
for the kernel's data which are related to PCI bus:

    :~ # cat /sys/bus/pci/devices/0000\:01\:00.0/vendor 
    0x10de

    :~ # cat /sys/bus/pci/devices/0000\:01\:00.0/device 
    0x0de0



The only problem with the device section is that I cannot
tell which monitor I want to use. That's why there is another section
called "Screen" which is going to be the "link" between "Monitor" and "Device"
sections.



  ------------------------------------------------------------------------------
 |                                          |
 |    -----------      =================                    |
 |   |       |    |         |                   | The screen is just a visual
interface
 |   | Monitor 0 |    | Card0       |                   | used to combine a
monitor with a 
 |   |       |    |             |                   | device.
 |    -----------      =================                    |
 |       | |        ||||||||| |||                           |
 |      /   \      --------- ----         ###########           |
 |      =====      | PCIe 01:00.0 | <--->  # nouveau # <---> KERNEL     |
 |                 --------- ----         ###########           |
 |                                      |
 |          Screen0                         |
  -------------------------------------------------------------------------------


Notice, that we haven't yet mentioned how the card0 is connected with the
Monitor0.
That has been said, X Server doesn't know if the monitor we are talking about
is VGA-0, DVI-I-1 or HDMI-1. We let X Server (for now) to choose this
automatically.

Last but not least, we need a Layout Section. This section is used in order to
place our screens in position. For now own, we have only one Monitor, thus
we need to define this screen as the primary one.

        Screen      0  "Screen0" 0 0

                            ^            ^ ^
                            |            | |
     Primary ---------------             | | relative positions (x horizontal,
y vertical)
                              x ---------  | which means the upper left corner
of our display
                              y -----------

                

        

4. Start the X Server:
----------------------

In order to start X Server and tell it to use this file you have 2 ways:

    I. Either, place the file into the appropriate directory:

        :~> cp xorg.conf.new /etc/X11/xorg.conf

       and then start X server:

        :~> X


    II. Or, you simply can pass the configuration file as an parameter
        while starting the X Server (recommended):


        :~> X -config /root/xorg.conf.new


You are going to see a black screen (this is surprisingly good)
which means that X Server is running. Go again back to 'tty1'
and kill it by pressing "ctrl+c".


Now, view the logs:

    :~> vim /var/log/Xorg.0.log


First, let's see if X Server uses the same Output-names
as XRandR does:

    :~> grep "Output.*connected" /var/log/Xorg.0.log

        [  3409.088] (II) NOUVEAU(0): Output DVI-I-1 connected
        [  3409.088] (II) NOUVEAU(0): Output HDMI-1 connected
        [  3409.088] (II) NOUVEAU(0): Output VGA-1 connected


The answer here is: yes it does.
Do you remember that we didn't tell X Server what Output name is used
for our monitor? Let's see what X Server probing picked up automatically:

    [  7177.781] (II) NOUVEAU(0): Output DVI-I-1 using monitor section Monitor0
    [  7177.813] (II) NOUVEAU(0): Output HDMI-1 has no monitor section
    [  7177.845] (II) NOUVEAU(0): Output VGA-1 has no monitor section


So, X Server thinks that my Monitor0 is connected using DVI. Which is correct,
but
this is pure luck. Let's see the resolution that X chose:


    [  7177.945] (II) NOUVEAU(0): Output DVI-I-1 using initial mode 1152x864
    [  7177.945] (II) NOUVEAU(0): Output HDMI-1 using initial mode 1152x864
    [  7177.945] (II) NOUVEAU(0): Output VGA-1 using initial mode 1152x864


Hm, it makes sense (if you look my output from xrandr) that 1152x864 is the
higher resolution
supported by *all* of my displays.

Let's try to fire-up gnome and see what is going to be. To do that, we first
have to place the
configuration to the right place:

    :~> cp xorg.conf.new /etc/X11/xorg.conf

and go into graphical target (aka Runlevel 5):

    :~> telinit 5


However, I still get the same result as my first state (without "xorg.conf").
Let's make sure first that X Server is reading our configuration:

    [  7848.626] (==) Using config file: "/etc/X11/xorg.conf"

So, the file is the correct one.

    [  7848.627] (==) ServerLayout "TripleHead Setup with nouveau"
    [  7848.627] (**) |-->Screen "Screen0" (0)
    [  7848.627] (**) |   |-->Monitor "Monitor0"
    [  7848.627] (**) |   |-->Device "Card0"


So does the rest of the setup. Let's try to force Monitor0 to be the DVI-I-1
connection
and see what happens. Go into your "device section" and put this line:


    Option      "Monitor-DVI-I-1"   "Monitor0"

and see what happens. In order to test this, we are going to follow the same
procedure:

    1. Change /etc/X11/xorg.conf
        2. Go to 'runlevel 3'
        3. Start X Server by typing "X"
        4. Go to tty1 (press ctrl+alt+F1)
        5. Kill the X Server (pres ctrl+c)
        6. See the logs (vi /var/log/Xorg.0.log)
    7. Go to 'runlevel 5'

Hm, it didn't change. Same results.

5) Optimize xorg.conf
---------------------

Let's try to add 3 monitors, which are going to be like our 3 displays.



    Section "Monitor"
        Identifier   "Monitor0"
    EndSection

    Section "Monitor"
        Identifier   "Monitor1"
    EndSection

    Section "Monitor"
        Identifier   "Monitor2"
    EndSection


And let's connect the to corresponding hardware according to my setup:


    Section "Device"
        Identifier  "Card0"
        Driver      "nouveau"
        BusID       "PCI:1:0:0"
        Option      "Monitor-DVI-I-1"   "Monitor0"
        Option      "Monitor-HDMI-1"    "Monitor1"
        Option      "Monitor-VGA-1"     "Monitor2"
    EndSection


let's test. Same result (from Gnome) but the logs have changed:


    [  8893.423] (II) NOUVEAU(0): Output DVI-I-1 using monitor section Monitor0
    [  8893.455] (II) NOUVEAU(0): Output HDMI-1 using monitor section Monitor1
    [  8893.487] (II) NOUVEAU(0): Output VGA-1 using monitor section Monitor2

Which is the correct configuration:


          ----------   ----------   ----------  
         |      | |      | |          |
         | Monitor0 | | Monitor1 | | Monitor2 |
         |      | |      | |          |
          ----------   ----------   ----------
             | |      | |          | |
             | |      | |          | |
            /   \    /   \        /   \
            =====    =====        =====

            DVI          HDMI          VGA

              |           |     |
               ------     |    ---------
                 \    |   /
                  \   |  /
                 ===========
                | NVIDIA    |
                | GeForce   |
                 ===========


Checking if they are all connected:

    [  8893.587] (II) NOUVEAU(0): Output DVI-I-1 connected
    [  8893.587] (II) NOUVEAU(0): Output HDMI-1 connected
    [  8893.587] (II) NOUVEAU(0): Output VGA-1 connected

They are all connected but still, I get:


          ----------   ----------   ----------  
         |      | |      | |          |
         | NoSignal | |  Mirror  | |  Mirror  |
         |      | |      | |          |
          ----------   ----------   ----------
             | |      | |          | |
             | |      | |          | |
            /   \    /   \        /   \
            =====    =====        =====

            DEAD           |        |
                   |        |
                    ------------ 
                  Mirrored by default


Check the logs:

    [  8893.587] (II) NOUVEAU(0): Output DVI-I-1 using initial mode 1152x864
    [  8893.587] (II) NOUVEAU(0): Output HDMI-1 using initial mode 1152x864
    [  8893.587] (II) NOUVEAU(0): Output VGA-1 using initial mode 1152x864


Well, let's try to disable the Mirrored-ones (Monitor1 and Monitor2)
by putting this line into their "Monitor Sections"

    Option       "Ignore"   "True"

So, it will look like this:

    Section "Monitor"
        Identifier   "Monitor0"
    EndSection

    Section "Monitor"
        Identifier   "Monitor1"
        Option       "Ignore"   "True"
    EndSection

    Section "Monitor"
        Identifier   "Monitor2"
        Option       "Ignore"   "True"
    EndSection


and continue testing....

Voila! For the first time, now, X Server does what we told him to do.


          ----------   ----------   ----------  
         |      | |      | |          |
         | Working  | | Disabled | | Disabled |
         |      | |      | |          |
          ----------   ----------   ----------
             | |      | |          | |
             | |      | |          | |
            /   \    /   \        /   \
            =====    =====        =====

           Monitor0     Monitor1     Monitor2
            DVI          HDMI          VGA


let's see what the logs has to say about this new change:

    [  9346.246] (II) NOUVEAU(0): Output DVI-I-1 using monitor section Monitor0
    [  9346.278] (II) NOUVEAU(0): Output HDMI-1 using monitor section Monitor1
    [  9346.278] (**) NOUVEAU(0): Option "Ignore" "True"
    [  9346.310] (II) NOUVEAU(0): Output VGA-1 using monitor section Monitor2
    [  9346.310] (**) NOUVEAU(0): Option "Ignore" "True"


which is correct (known from the previous configuration), but notice
that HDMI-1 and VGA-1 are now marked with the "Ignore" option.


    [  9346.346] (II) NOUVEAU(0): Output DVI-I-1 connected

Which has as a result that only DVI-I-1 to be connected. 

    [  9346.346] (II) NOUVEAU(0): Output DVI-I-1 using initial mode 1680x1050    

and as long as this is the only display, X server picked up the best resolution
supported
by my monitor (which makes sense because there is no other other monitor to
operate with).


So, let's try to leave only the HDMI disabled and see what is going to be. To
do that
remove the "Ignore" line from Monitor1.


    Section "Monitor"
        Identifier   "Monitor0"
    EndSection

    Section "Monitor"
        Identifier   "Monitor1"
    EndSection

    Section "Monitor"
        Identifier   "Monitor2"
        Option       "Ignore"   "True"
    EndSection

and Test

Works! Here's the expected results:


          ----------   ----------   ----------  
         |      | |      | |          |
         | Primary  | | 2ndary   | | Disabled |
         |      | |      | |          |
          ----------   ----------   ----------
             | |      | |          | |
             | |      | |          | |
            /   \    /   \        /   \
            =====    =====        =====

           Monitor0     Monitor1     Monitor2
            DVI          HDMI          VGA



let's see the logs:

    [ 10227.375] (II) NOUVEAU(0): Output DVI-I-1 using monitor section Monitor0
    [ 10227.407] (II) NOUVEAU(0): Output HDMI-1 using monitor section Monitor1
    [ 10227.439] (II) NOUVEAU(0): Output VGA-1 using monitor section Monitor2
    [ 10227.439] (**) NOUVEAU(0): Option "Ignore" "True"

That's correct. Only the Monitor2 (VGA) is disabled.


[ 10227.507] (II) NOUVEAU(0): Output DVI-I-1 connected
[ 10227.507] (II) NOUVEAU(0): Output HDMI-1 connected

Again, that's correct. The VGA-1 is disconnected ;)

[ 10227.507] (II) NOUVEAU(0): Using exact sizes for initial modes
[ 10227.507] (II) NOUVEAU(0): Output DVI-I-1 using initial mode 1680x1050
[ 10227.507] (II) NOUVEAU(0): Output HDMI-1 using initial mode 1680x1050

and also, my two monitors seems to have same resolution.

So, dual-head works. What's happens and thins go crazy when I am using 3
connected monitors.
So, far to my understaning is that the only thing that changes is the
placement. When
we have two monitors, it's pretty normal that one is going to be "left" and the
other
is going to be "right". For my current configuration, the primary monitor is
the left one,
which is monitor0. This can be also verified from gnome:

    Monitor0 ---> Primary
    Monitor1 ---> Secondary

so, if I need to place a third monitor, maybe we need to statically say to X
Server
where my third monitor is going to be:

    Section "Monitor"
        Identifier   "Monitor0"
    EndSection

    Section "Monitor"
        Identifier   "Monitor1"
    EndSection

    Section "Monitor"
        Identifier   "Monitor2"
        Option       "RightOf"  "Monitor1"


let's test it

Well, not exactly what I was expected:

          ----------   ----------   ----------  
         |      | |      | |          |
         | NoSignal | | Primary  | | 2ndary   |
         |      | |      | |          |
          ----------   ----------   ----------
             | |      | |          | |
             | |      | |          | |
            /   \    /   \        /   \
            =====    =====        =====

           Monitor0     Monitor1     Monitor2
            DVI          HDMI          VGA

let's see what the logs have to say about this new thing:

    [ 10778.986] (II) NOUVEAU(0): Output DVI-I-1 using monitor section Monitor0
    [ 10779.018] (II) NOUVEAU(0): Output HDMI-1 using monitor section Monitor1
    [ 10779.050] (II) NOUVEAU(0): Output VGA-1 using monitor section Monitor2
    [ 10779.050] (**) NOUVEAU(0): Option "RightOf" "Monitor1"

which is correct.


    [ 10779.151] (II) NOUVEAU(0): Output DVI-I-1 connected
    [ 10779.151] (II) NOUVEAU(0): Output HDMI-1 connected
    [ 10779.151] (II) NOUVEAU(0): Output VGA-1 connected

which is correct

    [ 10779.151] (II) NOUVEAU(0): Using user preference for initial modes
    [ 10779.151] (II) NOUVEAU(0): Output DVI-I-1 using initial mode 1680x1050
    [ 10779.151] (II) NOUVEAU(0): Output HDMI-1 using initial mode 1680x1050
    [ 10779.151] (II) NOUVEAU(0): Output VGA-1 using initial mode 1600x900

My third monitor uses different resolution (doesn't supprot 1680x1050).
So, as a last chance let's try to say the place for its display except from
the primary one (monitor0).

    Section "Monitor"
        Identifier   "Monitor0"
    EndSection

    Section "Monitor"
        Identifier   "Monitor1"
        Option       "RightOf"  "Monitor0"
    EndSection

    Section "Monitor"
        Identifier   "Monitor2"
        Option       "RightOf"  "Monitor1"
    EndSection




AAnd test...

same result....


After a *lot* of hacking into the "xorg.conf" file I didn't manage to enable
all three monitors.
The problem seems to be the CRTC. Most hardware only have 2 CRTCs, which means
that they have only
2 rendering pipelines. Simply put, only two 2 displays output ports can be
enabled at the same time.
I can see the CRTC using xrandr:

    :~> xrandr --verbose | grep CRTC

        CRTCs:      0 1
        CRTC:       0
        CRTCs:      0 1
        CRTC:       1
        CRTCs:      0 1

Which means:

    Monitor0 (DVI)  --> Supports: 0 and 1 --> Enabled: none , so that's why it
is turned off
    Monitor1 (HDMI) --> Supports: 0 and 1 --> Enabled: 0 (Working)
    Monitor1 (VGA)  --> Supports: 0 and 1 --> Enabled: 1 (Working)

This is why X Server disabled one display out of three, because there's no
enough CRTCs to use.
Thus, we are doomed to use only 2 monitors.


6. DualHead Setup:
~~~~~~~~~~~~~~~~~~

Since I am going to use only two monitors, I will pick the monitors that they
have
the same resolution. But before configure and visual screen (shared window),
let's enable them:



          ----------   ----------   ----------  
         |      | |      | |          |
         | Primary  | | Secondary| |   Off    |
         |      | |      | |          |
          ----------   ----------   ----------
             | |      | |          | |
             | |      | |          | |
            /   \    /   \        /   \
            =====    =====        =====

           Monitor0     Monitor1     Monitor2
            DVI          HDMI          VGA



This is the setup we I am going to start with:

    Section "ServerLayout"
        Identifier     "DualHead Setup with nouveau"    # Let's change the name
from triple to dual
        Screen      0  "Screen0" 0 0
    EndSection

    Section "Monitor"
        Identifier   "Monitor0"
        Option       "Primary"  "True"          # Force Monitor0 to be the
Primary one
    EndSection

    Section "Monitor"
        Identifier   "Monitor1"
        Option       "RightOf"  "Monitor0"      # This is going to be the
secondary display
    EndSection

    Section "Monitor"
        Identifier   "Monitor2"
        Option       "RightOf"  "Monitor1"
        Option       "Ignore"   "True"          # Turn this off (no CRTC left)
    EndSection

    Section "Device"
        Identifier  "Card0"
        Driver      "nouveau"
        BusID       "PCI:1:0:0"
        Option      "Monitor-DVI-I-1"   "Monitor0"
        Option      "Monitor-HDMI-1"    "Monitor1"
        Option      "Monitor-VGA-1"     "Monitor2"
    EndSection


    Section "Screen"
        Identifier "Screen0"
        Device     "Card0"
        Monitor    "Monitor0"
    EndSection


and let's test.

And it works. The result is as expected:


          ----------   ----------   ----------  
         |      | |      | |          |
         | Primary  | | Secondary| |   Off    |
         |      | |      | |          |
          ----------   ----------   ----------
             | |      | |          | |
             | |      | |          | |
            /   \    /   \        /   \
            =====    =====        =====

           Monitor0     Monitor1     Monitor2
            DVI          HDMI          VGA



let's see the logs:


    [ 66561.307] (II) NOUVEAU(0): Output DVI-I-1 using monitor section Monitor0
    [ 66561.307] (**) NOUVEAU(0): Option "Primary" "True"
    [ 66561.339] (II) NOUVEAU(0): Output HDMI-1 using monitor section Monitor1
    [ 66561.339] (**) NOUVEAU(0): Option "RightOf" "Monitor0"
    [ 66561.371] (II) NOUVEAU(0): Output VGA-1 using monitor section Monitor2
    [ 66561.371] (**) NOUVEAU(0): Option "RightOf" "Monitor1"
    [ 66561.371] (**) NOUVEAU(0): Option "Ignore" "True"
    ...
    ...
    ...
    [ 66561.440] (II) NOUVEAU(0): Output DVI-I-1 connected
    [ 66561.440] (II) NOUVEAU(0): Output HDMI-1 connected


which is correct. We had to disable the 3rd screen (Monitor1) because our
hardware
supports only 2 CRTCS:

    [ 66561.371] (II) NOUVEAU(0): 2 crtcs needed for screen.
    [ 66561.375] (II) NOUVEAU(0): Allocated crtc nr. 0 to this screen.
    [ 66561.375] (II) NOUVEAU(0): Allocated crtc nr. 1 to this screen.

Indeed, X Server confirms that 2 CRTCs are needed for the screen:

            ------------------------
    CRTC 0 --->    |            | 
    CRTC 1 --->    |  Monitor0     Monitor1 |
               |            |
            ------------------------
                      | |
                  | |
                 /   \
                 =====


Let's see what resolution is picked from X automatically:

[ 66561.440] (II) NOUVEAU(0): Using user preference for initial modes
[ 66561.440] (II) NOUVEAU(0): Output DVI-I-1 using initial mode 1680x1050
[ 66561.440] (II) NOUVEAU(0): Output HDMI-1 using initial mode 1680x1050

As expected, 1680x1050 is the top resolution for each of my individual
displays,
thus X server picked this up for both of them. 

The "trick" I love in this configuration is that we are using only one screen,
called "Screen0". This config, forced X Server to automatically combine
my two monitors, into the same screen, and not separate. Simply put,
there's only one DISPLAY env variable for both of my displays, which means
that I can share windows (moving them from one display to another).
The technical term for this Screen is called "Visual Screen", which
unifies the monitors into a greater resolution:


                1680 + 1680 = 3360
        -----------------------------
           |  -----------   -----------  |
           | |       | |       | |
           | | 1680x1050 | | 1680x1050 | |
           | |       | |       | |
           |  -----------   -----------  |
           |     | |       | |       |
           |     | |       | |       |
           |    /   \     /   \      |
           |    =====     =====      |
            -----------------------------
               |   |
               |   |
              /     \
             /   \
             =========
   
                   Visual Screen 3360x1050


So, let's see if my calculations are related to logs:

    [ 66561.440] (--) NOUVEAU(0): Virtual size is 3360x1050 (pitch 0)

indeed they are :)
So, the the X Server picked the correct mode (resolution + refresh rate) and
combined them altogether:

    [ 66561.440] (**) NOUVEAU(0):  Driver mode "1680x1050": 146.2 MHz (scaled
from 0.0 MHz), 65.3 kHz, 60.0 Hz
    [ 66561.440] (II) NOUVEAU(0): Modeline "1680x1050"x60.0  146.25  1680 1784
1960 2240  1050 1053 1059 1089 -hsync +vsync (65.3 kHz eP)


If the X Server has problem detecting the best mode, you have to explicitely,
set it up into the "xorg.conf"
But first you need to know the complete list of modes that your display
supports:

    :~ # xrandr 

        DVI-I-1 connected primary 1680x1050+0+0 (normal left inverted right x
axis y axis) 473mm x 296mm
           1680x1050     59.88*+
           1280x1024     75.02    60.02  
           1152x864      75.00  
           1024x768      75.08    60.00  
           800x600       75.00    60.32  
           640x480       75.00    60.00  
           720x400       70.08 
 
        HDMI-1 connected 1680x1050+1680+0 (normal left inverted right x axis y
axis) 473mm x 296mm
           1680x1050     59.95*+
           1280x1024     75.02    60.02  
           1152x864      75.00  
           1024x768      75.08    60.00  
           800x600       75.00    60.32  
           640x480       75.00    60.00  
           720x400       70.08


So, for my displays, the best mode for them to work together is 1680x1050@59.95
In order to generate the "modeline" I am going to use the "gtf" tool:

    gtf 1680 1050 59.95

      # 1680x1050 @ 59.95 Hz (GTF) hsync: 65.17 kHz; pclk: 147.01 MHz
      Modeline "1680x1050_59.95"  147.01  1680 1784 1968 2256  1050 1051 1054
1087  -HSync +Vsync

and then you have to put this info in your "xorg.conf". For my case this
optional, because
X Server already pick this modeline by itself, but it not quite optimized:


    X Server    ---> Modeline "1680x1050"x60.0  146.25  1680 1784 1960 2240
1050 1053 1059 1089 -hsync +vsync (65.3 kHz eP)
    gtf version ---> Modeline "1680x1050_59.95"  147.01  1680 1784 1968 2256
1050 1051 1054 1087  -HSync +Vsync

So, let's put the "gtf" result into my Xorg and see what happens:


    Section "ServerLayout"
        Identifier     "DualHead Setup with nouveau"
        Screen      0  "Screen0" 0 0
    EndSection

    Section "Monitor"
        Identifier   "Monitor0"
        Option       "Primary"  "True"
        Modeline     "1680x1050_59.95"  147.01  1680 1784 1968 2256  1050 1051
1054 1087  -HSync +Vsync # The gtf version
        Option       "PreferredMode"    "1680x1050_59.95"
# Tell X to pick the gtf version
    EndSection

    Section "Monitor"
        Identifier   "Monitor1"
        Option       "RightOf"  "Monitor0"
        Modeline     "1680x1050_59.95"  147.01  1680 1784 1968 2256  1050 1051
1054 1087  -HSync +Vsync # same
        Option       "PreferredMode"    "1680x1050_59.95"
# same
    EndSection

    Section "Monitor"
        Identifier   "Monitor2"
        Option       "RightOf"  "Monitor1"
        Option       "Ignore"   "True"
    EndSection

    Section "Device"
        Identifier  "Card0"
        Driver      "nouveau"
        BusID       "PCI:1:0:0"
        Option      "Monitor-DVI-I-1"   "Monitor0"
        Option      "Monitor-HDMI-1"    "Monitor1"
        Option      "Monitor-VGA-1"     "Monitor2"
    EndSection


    Section "Screen"
        Identifier "Screen0"
        Device     "Card0"
        Monitor    "Monitor0"

        SubSection "Display"        # Define the Virtual screen
                  Depth     24          # 24 is kind of a default nowadays
          Virtual   3360 1050       # The visual screen resolution
        EndSubSection

    EndSection


And test it... 

All good! Let's see the logs:

    [ 74689.115] (II) NOUVEAU(0): Using user preference for initial modes
    [ 74689.115] (II) NOUVEAU(0): Output DVI-I-1 using initial mode
1680x1050_59.95
    [ 74689.115] (II) NOUVEAU(0): Output HDMI-1 using initial mode
1680x1050_59.95

Ah so, the X server picked our preference and used our modeline. Great :D


    [ 74689.115] (--) NOUVEAU(0): Virtual size is 3360x1050 (pitch 0)
    [ 74689.115] (**) NOUVEAU(0):  Mode "1680x1050_59.95": 147.0 MHz (scaled
from 0.0 MHz), 65.2 kHz, 59.9 Hz
    [ 74689.115] (II) NOUVEAU(0): Modeline "1680x1050_59.95"x59.9  147.01  1680
1784 1968 2256  1050 1051 1054 1087 -hsync +vsync (65.2 kHz UP)


Here you can compare 1:1 the Modeline that X Server is using now, which is
identicall to the "gtf".






