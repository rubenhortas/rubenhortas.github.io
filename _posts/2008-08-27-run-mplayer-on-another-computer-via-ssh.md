---
title: Run mplayer on another computer via ssh
date: 2008-08-27 00:00:01 +0000
categories: [mplayer, ssh]
tags: [mplayer, ssh]
---
Let's suppose we are using a computer A ~~which is a laptop~~ and ~~we are lying in bed and~~ for some reason we want to watch a video on another computer B which ~~is our desktop and~~ is located elsewhere and is only accessible via ssh. How can we watch a video (in this case in full screen) on the computer B?
Once connected via ssh we run:

```console
$ DISPLAY=:0 mplayer -fs video
```

_Enjoy! ;)_
