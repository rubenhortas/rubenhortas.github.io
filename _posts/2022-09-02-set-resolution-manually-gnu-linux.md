---
title: Set a resolution manually in GNU/Linux
date: 2022-09-02 00:00:01 +0000
categories: [gnu/linux, screen resolution]
tags: [gnu/linux, screen, resolution ]
---

Lately I'm having problems with mi secondary screen. For some reason it loses its resolution and I have to add it manually.

To add a resolution manually in GNU/Linux we need:

* Generate a modeline for our resolution

```shell
cvt 1680 1050 60
```

This will show the following result

```
# 1680x1050 59.95 Hz (CVT 1.76MA) hsync: 65.29 kHz; pclk: 146.25 MHz
Modeline "1680x1050_60.00"  146.25  1680 1784 1960 2240  1050 1053 1059 1089 -hsync +vsync
```

* Add the modeline with xrandr:

```shell
xrandr --newmode "1680x1050_60"  146.25  1680 1784 1960 2240  1050 1053 1059 1089 -hsync +vsync
```

* Identify our screens

```shell
xrandr -q
```

This will show an output like:

```
Screen 0: minimum 320 x 200, current 3360 x 1050, maximum 8192 x 8192
DVI-I-1 connected primary 1680x1050+0+0 (normal left inverted right x axis y axis) 434mm x 270mm
   1680x1050     59.88*+  59.95
...
DVI-I-2 connected 1680x1050+1680+0 (normal left inverted right x axis y axis) 640mm x 360mm
   1360x768      59.80 +
...
```

Our screens are DVI-I-1 and DVI-I-2. In this case we will add the resolution to DVI-I-2.

* Add it to our screen

```shell
xrandr --addmode DVI-I-2 1680x1050_60
```

* Set the resolution to our screen

```shell
xrandr --output DVI-I-2 --mode 1680x1050_60
```

The changes will be lost after reboot. To keep the resolution permanently we can create the file ~/.xprofile with the following content:

```
#!/bin/bash

# Generate modeline
# cvt 1680x1050 60

xrandr --newmode "1680x1050_60"  146.25  1680 1784 1960 2240  1050 1053 1059 1089 -hsync +vsync
xrandr --addmode DVI-I-2 1680x1050_60
xrandr --output DVI-I-2 --mode 1680x1050_60
```

_Enjoy! ;)_
