---
title: Configuring awesome window manager
date: 2022-05-01 00:00:01 +0000
categories: [awesome, howtos]
tags: [awesomewm]
---

My favorite tiling manager is [awesome window manager](https://awesomewm.org/) (also called awesome or awesomewm). 
Awesome does not need mouse, everything can be performed with the keyboard and this is great when you are using a laptop.

Awesome comes complete with its own status bar and can handle a lot of things by default, but uses Lua scripting language for its configuration and , at first, could be a little messy.

So where do we start to configure it?

# Preparing the files
We will store our configuration file and theme under the .config folder in our /home

```shell
$ mkdir ~/.config/awesome    
$ mkdir ~/.config/awesome/themes
$ mkdir ~/.config/awesome/themes/ruben
```

Now, we will copy the awesome configuration file and theme to edit them and configure awesome to our liking.

```shell
$ sudo cp /etc/xdg/awesome/rc.lua ~/.config/awesome
$ sudo cp /usr/share/awesome/themes/default/theme.lua ~/.config/awesome/themes/ruben
```

# Configuring awesome

In this section we will edit the ~/.config/awesome/rc.lua file.

## Window layouts
I only use the maximized layout, so in the section /usr/share/awesome/themes/default/theme.lua I comment all the others


```
 awful.layout.layouts = {
     -- awful.layout.suit.floating,
     -- awful.layout.suit.tile,
     -- awful.layout.suit.tile.left,
     -- awful.layout.suit.tile.bottom,
     -- awful.layout.suit.tile.top,
     -- awful.layout.suit.fair,
     -- awful.layout.suit.fair.horizontal,
     -- awful.layout.suit.spiral,
     -- awful.layout.suit.spiral.dwindle,
      awful.layout.suit.max,
     -- awful.layout.suit.max.fullscreen,
     -- awful.layout.suit.magnifier,
     -- awful.layout.suit.corner.nw,
     -- awful.layout.suit.corner.ne,
     -- awful.layout.suit.corner.sw,
     -- awful.layout.suit.corner.se,
 }

```

## Tags

In this section we will configure the number of tags we want and their text.
I only use 4 tags, and, really, I change the numbers for [hack nerd font](https://github.com/ryanoasis/nerd-fonts/releases/download/v2.1.0/Hack.zip) characters.

```
awful.tag({ "1", "2", "3", "4"}, s, awful.layout.layouts[1])
```

## Menu bar launcher

I don't use the menu bar from the wibar, but I left it in the mouse, so I comment in the left widgets section:

```
{ -- Left widgets
    layout = wibox.layout.fixed.horizontal,
    --mylauncher,
    s.mytaglist,
    s.mypromptbox,
},
```

## Right widgets
First of all, we will comment all the widgets that will not use, then we can add ours.
I don't need to see the keyboard layour nor the layout box used (I only used maximized), so I comment them:

```
{ -- Right widgets
     layout = wibox.layout.fixed.horizontal,
     -- mykeyboardlayout,
     wibox.widget.systray(),
     mytextclock,
     -- s.mylayoutbox,
},

```

## Custom key bindings

One of the advantages to use a tiling window manager is have our custom keybindings, so I always add a few. 
For example, if we want a key binding to firefox, we will edit the {{{ Key bindings section and add:

```
awful.key({ modkey }, "f", function() awful.util.spawn("firefox") end,
          {description = "firefox", group = "applications"})
```
* Be careful in this step. The previous entry has to end with a colon, and the last entry in this section has to end without colon.

# Configuring our theme

In this section we will edit the our theme file, in my case: ~/.config/awesome/themes/ruben/theme.lua

## Font

I find the default font a bit small, so I set one a little bit bigger:

```
theme.font = "sans 12"
```


## Wallpaper

We can set our wallpaper: 

```
theme.wallpaper = themes_path.."ruben/background.jpg"

```

If we don't have our wallpaper on the themes path, we can set it with adding its absolute path:

```
theme.wallpaper = themes_path.."/home/ruben/images/wallpaper.jpg"

```


When we have awesome running configured to our liking, we can make a backup of our configuration files and themes, then we can remove the commented lines and their associated sections and/or declarations. This way we will have a (slightly) lighter configuration and window manager and, with a backup, we can always go back in case something goes wrong.

_Enjoy! ;)_
