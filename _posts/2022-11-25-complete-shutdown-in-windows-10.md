---
title: Fixing awesome logout-menu-widget
date: 2022-11-18 00:00:01 +0000
categories: [awesome, widget]
tags: [awesome, widget, logout-menu-widget, howtos]
---

I like to have installed the [logout-menu-widget](https://github.com/streetturtle/awesome-wm-widgets/tree/master/logout-menu-widget) from [Pavel Makhov](https://pavelmakhov.com/) ([@streetturtle](https://github.com/streetturtle))in awesome. 
I like to can shutdown my machine with a pair of clicks.
But, it turns out that the [logout-menu-widget](https://github.com/streetturtle/awesome-wm-widgets/tree/master/logout-menu-widget) does not work for me out of the box.
It's just a configuration problem, and to solve it I have to change a couple of lines in the widget source code (logout-menu.lua file).

## Fixing the icon

The first problem that arises is that the icon does not appear.
It's just a path problem.
I have my widgets under a widget directory, so I have to change the path in the following line:

```lua
local ICON_DIR = HOME .. '/.config/awesome/awesome-wm-widgets/logout-menu-widget/icons/'
```

And set my personal path:

```lua
local ICON_DIR = HOME .. '/.config/awesome/widgets/logout-menu-widget/icons/'
```

## Fixing the shutdown

I don't have the shutdown alias defined for my user, so the shutdown option does not work.
And the "now" argument does not work with my version of shutdown.
To fix this (without have to add a new alias) I change the command in the following line:

```lua
local onpoweroff = args.onpoweroff or function() awful.spawn.with_shell("shutdown now") end
```

And set the absolute path for the shutdown binary and the options to shutdown the system immediately:

```lua
local onpoweroff = args.onpoweroff or function() awful.spawn.with_shell("/sbin/shutdown -h now") end
```

## Disabling options

Since I am already editing the file, and since I never use the suspend option, I take this oportuntity to disable the suspend option.
To do this i comment the following line:

```lua
-- { name = 'Suspend', icon_name = 'moon.svg', command = onsuspend },
```

And my widget is ready.

_Enjoy! ;)_
