---
title: Force HDMI hotplug in a Raspberry Pi
date: 2022-11-12 00:00:01 +0000
categories: [howtos, raspberry pi]
tags: [howtos, raspberry pi, hdmi] 
---

The HDMI output of the Raspberry Pi is only activated if a screen is connected and powered up before the Raspberry Pi is started up.
This means that if you have a Raspberry Pi connected to your tv, and your Raspberry Pi is rebooted while the tv is unplugged, the HDMI output of the Raspberry Pi will not be activated.
In this case you will need to restart your Raspberry Pi to be able to see it on your tv.

How can we avoid this?

We can force a HDMI hotplug on our Raspberri Pi.
In other words: we can configure our Raspberry Pi so that it activates the HDMI regardless of whether it has a screen connected.

To force the HDMI hotplug the first thing we will need is to known our HDMI mode and group.
With a screen connected to our Raspberry Pi we will execute:

```shell
$ tvservice -s
``` 

Based on the information returned to us we can set our values.
For example, if or output was:

```shell
state 0xa [HDMI CEA (31) RGB lim 16:9], 1920x1080 @ 50.00Hz, progressive
```

Based on our output, our values will be:
  - hdmi_group=1 # CEA 
  - hdmi_mode=31 # (CEA) = (31) = 1080p = 1920x1080, 16:9, 50Hz
  
_*We also can find (or check) our configuration values at [Raspberry Pi documentation video options](https://www.raspberrypi.com/documentation/computers/config_txt.html#video-options)_
  
Now we will need to edit the /boot/config.txt file (as root) and uncomment and set the following values:

```shell
hdmi_force_hotplug=1
hdmi_group=1
hdmi_mode=31
hdmi_drive=2 # Optional, this can make audio work in DMT (computer monitor) modes.
```

And the HDMI hotplug will work after reboot the Raspberry Pi.

_Enjoy! ;)_
