---
title: Hack The Box - misDIRection pwned!
date: 2022-05-05 00:00:01 +0000
categories: [hack the box, challenge]
tags: [hack the box, challenge, misc, htb, misdirection]
img_path: /assets/img/posts/
---

## Description

> During an assessment of a unix system the HTB team found a suspicious directory. They looked at everything within but couldn't find any files with malicious intent.

## Solution

When we extract the misDIRection.zip whe find the hidden folder .secret
The .secret folder contains a list of subfolders named as the numbers from 0-9 and letters from a-z and A-Z.

![.secret folder](misdirection.png)
*.secret folder* 

Every each subfolder is empty or contains a empty file named with a number.
Let's suppose that, if the subfolder it's not empty, the subfolder name will be the character in the flag and the contained file name will be the position on the flag string...

When we get the flag string we'll see that is encoded in base64 so we'll need to decode it to get the flag.

We can solve it by hand but, to automate it, I did a self-explained python script that you can see at [secret_finder.py](https://github.com/rubenhortas/hackthebox/blob/main/misDirection/secret_finder.py)

![Challenge results](owned-misdirection.png){: width="700" height="600" .shadow}
*misDIRection has been Pwned*

*Enjoy! ;)*
