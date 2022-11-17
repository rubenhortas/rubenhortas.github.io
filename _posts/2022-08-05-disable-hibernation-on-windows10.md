---
title: Disable hibernation on Windows10
date: 2022-08-05 00:00:01 +0000
categories: [windows10, administration]
tags: [windows, windows10, administration, howtos]
---

Hibernate is a power option that will allow you to turn off your computer but save all the work you had open for when it is turned on again. The downside is that hibernation uses some storage space on your computer. 

I have a laptop with a relative small HD, so each Gb counts. In my case the hibernation file is occuping about 4Gb. I've never used hibernation, and I don't think I'm going to start at this point, so why have a useless file occupying about 4Gb?

To disable completly hibernation on windows10:

Run cmd as administrator and execute:

```shell
powercfg.exe /hibernate off
```

_Enjoy! ;)_
