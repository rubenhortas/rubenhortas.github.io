---
title: Complete shutdown on windows 10
date: 2022-11-25 00:00:01 +0000
categories: [windows10, administration]
tags: [windows10, administration, windows, shutdown, howto, administration]
---

Sometimes I feel like the last person in the world to find out things.
This is one of this times...

Windows 10 it was released on 2015, and I still discovered these days that windows doesn't shut down completelly.

By default, when you choose the option "shut down" in Windows 10, Windows 10 doesn't shut down completelly. 
Windows 10 is doing a partial hibernation through a feature called "fast startup".

Fast startup is a hybrid shutdown. 
With fast startup enabled, instead of shutting down completely, the computer enters in a sleep state that will allow it to boot faster the next time.

The problem, besides the power consumption, is that a shutdown, a reboot and starting from a suspended state do not perform the same tasks.

# How to completelly shut down Windows 10

There are two ways to completely turn off the computer:
 
* Windows menu > hold the shift key while you click on "shut down"
* Disabling fast startup: Settings > System > Power Options > Choose what the power buttons do > Change settings that are currently unavailable > (Uncheck) Turn on fast startup (recommended)

If you want to know a little more about how shut down and reboot works, and its differences, I recommend you the following videos from [Dave's garage](https://www.youtube.com/c/DavesGarage):
* [The Windows Triple Fault: plus Restarting vs Rebooting - Why it Matters!](https://www.youtube.com/watch?v=E8gOW0hFoJ0)
* [You're Doing it Wrong: Rebooting! Find out why!](https://www.youtube.com/watch?v=lUIhzACQDAc)
* [Why Does Rebooting Fix Everything? Ask a Microsoft Engineer!](https://www.youtube.com/watch?v=9IPP39OF78M)

_Enjoy! ;)_