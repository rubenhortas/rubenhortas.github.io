---
title: Hack the box xorxorxor
date: 2022-05-01 00:00:01 +0000
categories: [hack the box, challenge]
tags: [hack the box, challenge, crypto, htb, xorxorxor, xor, kpa]
--

>Who needs AES when you have XOR?

When we extract the xorxorxor.zip file we find two files:
- challenge.py The python script used to encrypt and decrypt with the xor function
- output.txt The flag in hexadecimal

If we take a look at the challenge.py script we can see that the key used to encrypt/decrypt is a 4 bytes random key, so every time that we encrypt/decrypt the key will change.

```python
self.key = os.urandom(4)
```

Once we have the hexadecimal string converted to text the challenge is get the key used to encrypt the plain text flag.

This challenge is based on a [XOR known-plaintext attack (kpa)](https://en.wikipedia.org/wiki/Known-plaintext_attack).
I explained how to perform this kind of attack in [XOR known-plaintext attack](https://rubenhortas.github.io/posts/xor-known-plaintext-attack/)

To solve this challenge, I did a self-explanatory python script that you can see here: [Unxorxorxor.py]()

_Enjoy! ;)_
