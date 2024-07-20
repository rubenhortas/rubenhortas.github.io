---
title: Hack the box Canvas
date: 2022-05-01 00:00:01 +0000
categories: [hack the box, challenge]
tags: [hack the box, challenge, misc, htb, canvas, javascript]
img_path: /assets/img/posts/
--

## Challenge description

>We want to update our website but we are unable to because the developer who coded this left today. Can you take a look?

## Solution

Wen we extract the Canvas.zip we found the next files:
- css folder
- js folder
- dashboard.html
- index.html

Let's take a look at the js folder. There is a login.js file, sounds good.  Let's take a look inside...  
The file is obfuscate but the last line contains a variable called res... res as in result? It can't be that easy...  

```js
var res=String['\x66\x72\x6f\x6d\x43\x68\x61\x72\x43\x6f\x64\x65'](0x48,0x54,0x42,0x7b,0x57,0x33,0x4c,0x63,0x30,0x6d,0x33,0x5f,0x37,0x30,0x5f,0x4a,0x34,0x56,0x34,0x35,0x43,0x52,0x31,0x70,0x37,0x5f,0x64,0x33,0x30,0x62,0x46,0x75,0x35,0x43,0x34,0x37,0x31,0x30,0x4e,0x7d,0xa);
```

Seems that is in hexadecimal, let's take the part of the strings between parentheses and check...

> (0x48,0x54,0x42,0x7b,0x57,0x33,0x4c,0x63,0x30,0x6d,0x33,0x5f,0x37,0x30,0x5f,0x4a,0x34,0x56,0x34,0x35,0x43,0x52,0x31,0x70,0x37,0x5f,0x64,0x33,0x30,0x62,0x46,0x75,0x35,0x43,0x34,0x37,0x31,0x30,0x4e,0x7d,0xa)

We clean the string removing the parentheses, the 0x and the commas:

>4854427b57334c63306d335f37305f4a3456343543523170375f6433306246753543343731304e7da

We check the result converting the string to text.  
This time I'll use the [hex string converter from codebeautify](https://codebeautify.org/hex-string-converter).

And we get our flag!

## Pwned

![Challenge results](owned-canvas.png){: width="700" height="600" .shadow}
*Canvas has been Pwned*

*Enjoy! ;)*
