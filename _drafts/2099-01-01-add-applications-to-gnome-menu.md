---
title: Add applications to gnome menu
date: 2022-05-01 00:00:01 +0000
categories: [gnome, howtos]
tags: [gnome, howtos, gnu/linux]
---

When we install applications from outside the repositories these applications are not added automatically to the gnome menu.  
How can we add them to the menu?

In [gnome](https://www.gnome.org/) applications are added to the menu through .desktop files.  
Desktop files are in two paths:  
* /usr/share/applications/
* ~/.local/share/applications

To add an application to the menu we need to create a .desktop file for the application in either of the two paths.  
For instance, if we want to add an entry in the menu for myNewApplication we will create myNewApplication.desktop file in one of the paths with the following content:

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
``` 

*We well need to adjust the values to our application.

_Enjoy! ;)_
