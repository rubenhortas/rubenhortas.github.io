---
title: Decrypting vigenere cipher
date: 2022-11-05 00:00:01 +0000
categories: [cryptography, vigenere]
tags: [cryptography, vigenere, overthewire, krypton, personal, developments] 
---

In my free time I like to play around in [CTF (Capture The Flag or Wargames)](https://en.wikipedia.org/wiki/Capture_the_flag_(cybersecurity)) platforms, learning a bit and, whith a bit of luck, solving some level or challenge.

One of the platforms in which I play and learn (when I can) is [OverTheWire](https://overthewire.org). 
[OverTheWire](https://overthewire.org) does not need registration and is one of the old good Wargames classics. 
In [OverTheWire](https://overthewire.org) you can learn about many topics at different levels (from 0, or almost).
The bad part of being online for so long is that there are a lot of walkthroughs out there.
The good part of being online for so long is that there are a lot of walkthroughs out there.

Some time  ago I started to solve the [Krypton](https://overthewire.org/wargames/krypton/) game from [OverTheWire](https://overthewire.org/wargames/).
This time it was time to solve [Level 4](https://overthewire.org/wargames/krypton/krypton4.html).
This level consists in decrypt a [Vigenere Cipher](https://en.wikipedia.org/wiki/Vigen%C3%A8re_cipher).
[Vigenere Cipher](https://en.wikipedia.org/wiki/Vigen%C3%A8re_cipher) is a polyalphabetic substitution cipher where one character of the plaintext (pt) may map to many, or all, possible ciphertext (ct) characters.
[Vigenere Cipher](https://en.wikipedia.org/wiki/Vigen%C3%A8re_cipher) it's like a sequence of several [Caesar ciphers](https://en.wikipedia.org/wiki/Caesar_cipher) with different shift values.

Although [Vigenere Cipher](https://en.wikipedia.org/wiki/Vigen%C3%A8re_cipher) dates from 1553 and today it's trivial to break it, we can found it in many cryptography challenges, even in the [CIA's Kryptos sculpture](https://en.wikipedia.org/wiki/Kryptos) ([Solution of passage 2](https://en.wikipedia.org/wiki/Kryptos#Solution_of_passage_2)).

Let's go back to the game...

At first I thought it wold be a waste of time. I mean, today it's trivial to break a [Vigenere Cipher](https://en.wikipedia.org/wiki/Vigen%C3%A8re_cipher).
You can get the [cipher] key, in a matter of seconds, using offline or online cryptography tools, for example: [dcode](https://www.dcode.fr/en).
But the purpose of this kind of challenges is to learn, not to get the key and run... So, although I knew the [Vigenere Cipher](https://en.wikipedia.org/wiki/Vigen%C3%A8re_cipher), and I knew how to encrypt using it and how to break it using tools, I didn't know how the process of breaking the encryption worked.
So I got curious and I started to research...

After a while I found a series of great videos in the youtube channel [Theoretically](https://www.youtube.com/user/ddxfraxinusdne):
* [Vigenere Cipher - Encryption](https://www.youtube.com/watch?v=izFivfLjD5E)
  This video is an introduction to the Vigenere Cipher.
* [Vigenere Cipher - Decryption (Known Key)](https://www.youtube.com/watch?v=oHcJ4QLiiP8)
  This video shows how to decrypt the ciphertext with a known key.
* [Vigenere Cipher - Decryption (Unknown Key)](https://www.youtube.com/watch?v=LaWp_Kq0cKs)
  This video shows how to find the key length (I didn't know at the time, but it would be useful for the [Krypton Level 5](https://overthewire.org/wargames/krypton/krypton5.html)) and explains how to find the key.

[@Theoretically](https://www.youtube.com/user/ddxfraxinusdne) explains very well and uses very simple examples so that people understands the concepts easily.
As the method seemed simple, I decided to give it a try and I started to coding a python script.
But, what seemed like a, more or less, quick task it ended not to being so.
Once started I found some blanks that I had to fill/solve in order to break the cipher and get the key, for example:

[Krypton Level 4](https://overthewire.org/wargames/krypton/krypton4.html)

* In [Vigenere Cipher - Decryption (Unknown Key) 11:46](https://youtu.be/LaWp_Kq0cKs?t=706) [@Theoretically](https://www.youtube.com/user/ddxfraxinusdne) says "__frecuencies of the numbers or the alphabet in order__".  
  Ok, but... In what order? As in his example the letters and the frecuencies are a:0.10, b:0.20 and c:0.70 all is in order. 
  The letters are in alphabetical order and the frecuencies are In increasing order...  
  Later I would find out that **the correct order is in alphabetical order**.

* Finding the key shifts  
  In [@Theoretically](https://www.youtube.com/user/ddxfraxinusdne)'s example the characters of the ciphertext (ct) match the plaintext (pt) alphabet characters (A,B,C and a,b,c).
  What do we do when some characters do not appear in our frecuency analysys? We skip them or we add them? Are they important?  
  Later I would find that, **in order to perform the rotations to find the key shifts we will need a full (sorted in alphabetical order) alphabet**.  
  So, once we have our ciphertext (ct) frecuency analysis, we need to complete the alphabet and assign a frecuency of 0 to the missing characters.

[Krypton Level 5](https://overthewire.org/wargames/krypton/krypton5.html)

* What is the best way to get the rotations with most coincidences to get the shift length?  
  I solved it calculating the median of the values of the rotation (directly with numpy).  
  It is the best idea I came up with.

* How to discard the false positives finding the key length?  
  Trying to finding the key length I found some false positives. But after a few debugging and thinking I found that was a logic error in programming.

I'm sure that I'm forgetting some doubts that arose while solving this challenge, but I think that these were the most important ones.
Answering these questions was a processs that involved more research and a programming method basedn on trial and error that involved a lot of debugging and thinking.

Was it worth it?

I'd like to think so. In the process of solving these challenges I was very entertained and I learned a few things, for example:

* I learned how to works the process to break a [Vigenere Cipher](https://en.wikipedia.org/wiki/Vigen%C3%A8re_cipher) through this method.
* I learned about other methods to break a [Vigenere Cipher](https://en.wikipedia.org/wiki/Vigen%C3%A8re_cipher), to find the key length and to find the key.
* I improved my python list slicing skills.

To solve the [Krypton](https://overthewire.org/wargames/krypton/) [Level 4](https://overthewire.org/wargames/krypton/krypton4.html) and [Krypton Level 5](https://overthewire.org/wargames/krypton/krypton5.html) I did a python script that you can see at [vigenere_decrypter.py](https://github.com/rubenhortas/python_examples/blob/master/cryptography/vigenere_decrypter.py)

_Enjoy! ;)_
