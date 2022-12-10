---
title: Swap two keys in GNU/Linux
date: 2022-12-22 00:00:01 +0000
categories: [debian, administration]
tags: [debian, administration, setxkbmap, keyboard]
---

I recently had to change my old worn pc105 keyboard.
One of the disadvantages of my new 60% keyboard is that I have to use its "mac layer" to have working the F keys.
And, in mac keyboards, the win/windows/supr/cmd/command (call if what you prefer)  key is swapped with the left alt key...

Well, this doesn't seem a big problem, but I use [awesome](https://awesomewm.org/), and, in [awesome](https://awesomewm.org/) the mod key is the windows key...
This means that the windows key is an essential key, and one of the most used keys, and I don't like to have it in another position.

So, I needed to remap my keboard and swap the win key and the left alt key.
Fortunately, it's an easy fix, and with setxkbmap, we can swap this two keys executing the following command:

```shell
setxkbmap -option altwin:swap_lalt_lwin
```

_Enjoy! ;)_
