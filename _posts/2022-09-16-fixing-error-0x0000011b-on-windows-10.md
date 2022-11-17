---
title: Fixing error 0x0000011b on Windows 10
date: 2022-09-16 00:00:01 +0000
categories: [windows10, administration]
tags: [windows, windows10, error, howtos, printer, administration]
---

A few days ago, as a personal favor, I had to configure some network printers between some Windows 10 computers. 
Nothing complicated... Until error 0x0000011b appeared...

> Windows cannot connect to the printer. Error: Operation failed with error 0x0000011b on the network.

## What is error 0x0000011b?

Error 0x0000011b is a bug that, in most cases, has occurred after the installation of a security patch that mitigates the [Windows Print Spooler Spoofing Vulnerability CVE-2021-1678](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2021-1678).

## How to Fix Printer Error 0x0000011b?

As it is a bug that occurs after a windows update, my first option was to update the machines.
In theory, to this day, Microsoft has fixed this bug, but it did not work for me.

So after a bit of googling I saw that other solutions were to uninstall a couple of updates and prevent them from being reinstalled or edit the windows registry to disable the security patch.

I ruled out the option to uninstall updates and set windows update not to install them again, basically for the following reasons:
* The user does not have an advanced profile and it is very possible that he will reinstall the updates by updating windows, and then he will be left without printers again...
* The time it would take to uninstall certain updates one by one and test if the error was fixed
* The chance of someone exploiting that vulnerability in that environment is very remote 
* The vulnerability does not seem critical

Once the windows update related options were discarded, the only thing left to do was to give it a try to edit the windows registry...

## Editing the windows registry to disable CVE-2021-1678 mitigation

* Run regedit.exe
* Navigate to HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\Print
* Create new New DWORD (32-bit) Value named RpcAuthnLevelPrivacyEnabled with value 0
* Reboot

And... Works for me!

_Enjoy! ;)_
