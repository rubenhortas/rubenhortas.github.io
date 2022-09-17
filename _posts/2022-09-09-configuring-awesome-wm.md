---
title: Configuring awesome window manager
date: 2022-09-09 00:00:01 +0000
categories: [awesome, howtos]
tags: [awesomewm, howtos, gnu/linux]
---

My favorite tiling windows manager is [awesome window manager](https://awesomewm.org/) (also called awesome or awesomewm). 
Awesome does not need mouse, everything can be performed with the keyboard and this is great when you are using a laptop.

Awesome comes complete with its own status bar and can handle a lot of things by default, but uses Lua scripting language for its configuration and, at first, could be a little messy.

So where do we start to configure it?

# Preparing the configuration files
We will store our configuration file and theme under the .config/ folder in our /home

```shell
$ mkdir ~/.config/awesome    
$ mkdir ~/.config/awesome/themes
$ mkdir ~/.config/awesome/themes/ruben
```

Now, we will copy the default awesome configuration file and theme to edit them and configure to our liking.

```shell
$ sudo cp /etc/xdg/awesome/rc.lua ~/.config/awesome
$ sudo cp /usr/share/awesome/themes/default/theme.lua ~/.config/awesome/themes/ruben
```

# Configuring awesome
In this section we will edit the ~/.config/awesome/rc.lua file.

## Setting the theme
We need to set our theme path as the parameter passed to beautitul.init:

```lua
beautiful.init("/home/ruben/.config/awesome/themes/ruben/theme.lua")
```

## Window layouts
I only use the maximized layout, so in the section /usr/share/awesome/themes/default/theme.lua I comment all the others:

```lua
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
I only use 2 tags in each screen, and, really, I change the numbers for [hack nerd font](https://github.com/ryanoasis/nerd-fonts/releases/download/v2.1.0/Hack.zip) characters.

```lua
awful.tag({ "1", "2" }, s, awful.layout.layouts[1])
```

## Menu bar launcher
I don't use the menu bar from the wibar, neither in the mouse (I prefer the Super+p or Super+r shortcuts), so I comment the awesome menu sections:

```lua
--myawesomemenu = {
--   { "hotkeys", function() hotkeys_popup.show_help(nil, awful.screen.focused()) end },
--   { "manual", terminal .. " -e man awesome" },
--   { "edit config", editor_cmd .. " " .. awesome.conffile },
--   { "restart", awesome.restart },
--   { "quit", function() awesome.quit() end },
--}
 
-- local menu_awesome = { "awesome", myawesomemenu, beautiful.awesome_icon }
-- local menu_terminal = { "open terminal", terminal }

-- if has_fdo then
--    mymainmenu = freedesktop.menu.build({
--        before = { menu_awesome },
--        after =  { menu_terminal }
--    })
--else
--   mymainmenu = awful.menu({
--        items = {
--                  menu_awesome,
--                  { "Debian", debian.menu.Debian_menu.Debian },
--                  menu_terminal,
--                }
--    })
--end
 
--mylauncher = awful.widget.launcher({ image = beautiful.awesome_icon,
--                                     menu = mymainmenu })
 
-- Menubar configuration
--menubar.utils.terminal = terminal -- Set the terminal for applications that require it
-- }}}
```

```lua
{ -- Left widgets
    layout = wibox.layout.fixed.horizontal,
    --mylauncher,
    s.mytaglist,
    s.mypromptbox,
},
```

```lua
-- root.buttons(gears.table.join(
--    awful.button({ }, 3, function () mymainmenu:toggle() end),
--    awful.button({ }, 4, awful.tag.viewnext),
--    awful.button({ }, 5, awful.tag.viewprev)
--))
```

```lua
--clientbuttons = gears.table.join(
--    awful.button({ }, 1, function (c)
--        c:emit_signal("request::activate", "mouse_click", {raise = true})
--    end),
--    awful.button({ modkey }, 1, function (c)
--        c:emit_signal("request::activate", "mouse_click", {raise = true})
--        awful.mouse.client.move(c)
--    end),
--    awful.button({ modkey }, 3, function (c)
--        c:emit_signal("request::activate", "mouse_click", {raise = true})
--        awful.mouse.client.resize(c)
--    end)
--)
```

## Right widgets
In this section we will comment all the widgets that will not use, later we can add ours.
I only want to keep the clock widget, so I comment the others:

```lua
{ -- Right widgets
     layout = wibox.layout.fixed.horizontal,
     -- mykeyboardlayout,
     -- wibox.widget.systray(),
     mytextclock,
     -- s.mylayoutbox,
},
```

## Custom key bindings
First of all I change the Super+j and Super+k keybinndings, is more intiutive this way for me.

One of the advantages to use a tiling window manager is have our custom shortcuts, so I always add a few. 
For example, if we want a shortcut to firefox, we will edit the Key bindings section and add:

```lua
awful.key({ modkey }, "f", function() awful.util.spawn("firefox") end,
          {description = "firefox", group = "internet"})
```
* Be careful in this step. The previous entry has to end with a colon, and the last entry in this section has to end without colon.

## Mapping applications to screen tags

If we want an application always running in a screen tag, we only have to set a rule.  
For example, if we want that MPlayer runs always in the tag 1 of screen 2:

```lua
{ rule = { class = "MPlayer" },
  properties = { screen = 2, tag = "1" } },
```

## Removing gaps 
For some reason, some windows, in maximized mode, have gaps at the right and bottom. To remove the gaps I add the "size_hints_honor = false" to the rules

```lua
{ rule_any = {type = { "normal", "dialog" }
  }, properties = { titlebars_enabled =  false, size_hints_honor = false }
},     
```

# Configuring our theme
In this section we will edit the our theme file, in my case: ~/.config/awesome/themes/ruben/theme.lua

## Font
I find the default font a bit small, so I change it and set one a little bit bigger:

```lua
theme.font = "Hack Nerd Font Mono Regular 9"
```

## Taglist font
I like to see the taglists very big, so I set their font to one bigger:

```lua
theme.taglist_font="Hack Nerd Font Mono Regular 16"
```

## Wallpaper
We can set our wallpaper editing the theme.wallpaper value:

```lua
theme.wallpaper = themes_path.."ruben/background.jpg"
```

If we don't have our wallpaper on the themes_path, we can set it with adding its absolute path:

```lua
theme.wallpaper = themes_path.."/home/ruben/images/wallpaper.jpg"
```

# Widgets

We can add our widgets to end the configuration. A good startingp point is [https://github.com/streetturtle/awesome-wm-widgets](https://github.com/streetturtle/awesome-wm-widgets)
I always install the following widgets:
  * cpu_widget
  * ram_widget
  * fs_widget

You also may find interesting my own awesome widgets:
  * [awesome ip widget](https://github.com/rubenhortas/awesome-ip-widget)
  * [awesome htb widget](https://github.com/rubenhortas/awesome-htb-widget)

# GTK applications  with dark theme
For personal reasons I usually work with dark themes, but awesome does not have one, is not its job.  
If you, as me, want applications in dark mode and you have installed gtk applications, you only have to modify the settings.ini files contained in the gtk-\*.\*/ directories under ~./config and set:

```
gtk-application-prefer-dark-theme=1
```

# Default browser
If we have many browsers installed and we want to use one of them as default browser, for instance firefox, we will find that some applications will open the links in another browser.
To solve this we will need to set the default browser, system-wide and user-specific. For instance, in debian:

## System-wide

```shell
$ sudo update-alternatives --config x-www-browser
```

## User-specific

```shell
$ xdg-settings set default-web-browser firefox-esr.desktop
```

# Backup and clean
When we have awesome running configured to our liking, we can make a backup of our configuration files and themes, then we can remove the commented lines and their associated sections and/or declarations. This way we will have a (slightly) lighter configuration and window manager and, with our backup, we can always go back in case something goes wrong.

# Further readings
* [My first awesome](https://awesomewm.org/doc/api/documentation/07-my-first-awesome.md.html)
* [awesome - ArchWiki](https://wiki.archlinux.org/title/awesome)
* [awesome recipes](https://awesomewm.org/recipes/)

_Enjoy! ;)_
