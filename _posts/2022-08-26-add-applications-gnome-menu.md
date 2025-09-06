---
title: Add applications to gnome menu
date: 2022-08-26 00:00:01 +0000
categories: [gnu/linux, gnome]
tags: [gnu/linux, gnome, configuration]
---

When we install applications from outside the repositories these applications are not added automatically to the gnome menu.
How can we add them to the menu?

In [gnome](https://www.gnome.org/) applications are added to the menu through .desktop files.

.desktop files are in two paths:
* /usr/share/applications/
* ~/.local/share/applications

To add an application to the menu we need to create a .desktop file for the application in either of the two paths.

For instance, if we want to add an entry in the menu for myNewApplication we will create myNewApplication.desktop file in one of the paths with the following content (setting the values for our application):


```
[Desktop Entry]
Version=1.0.0
Type=Application
Name=My new application
Icon=/path/to/mynewapplication/images/icon.svg
Exec=/path/to/mynewapplication/bin/mynewapplication.sh
Comment=My new application
Categories=Category1;Category2
Terminal=false
StartupWMClass=jetbrains-pycharm-ce
StartupNotify=true
```

## StartupWMClass

StartupWMClass is a property to associate windows with the owning application. Avoids to have multiple copies of the same icon on the dock for many java-based applications.
To find out which WM_CLASS has our application, we can run xprop on a terminal an then click at the application window.

```shell
xprop WM_CLASS
```

_Enjoy! ;)_
