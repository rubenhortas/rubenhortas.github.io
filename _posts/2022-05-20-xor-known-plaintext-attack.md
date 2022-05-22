---
title: XOR known-plaintext attack
date: 2022-05-20 00:00:01 +0000
categories: [security, xor]
tags: [vulnerabilities, xor, security, personal]
img_path: /assets/img/posts/
---

[XOR cipher](https://en.wikipedia.org/wiki/XOR_cipher) is a type of additive cipher extremely common as a component in more complex ciphers.  

XOR cipher can trivially be broken using frequency analysis, and, if the content of any message can be guessed or, otherwise, known then the key
then can be revealed.

The XOR cipher is vulnerable to a [**known-plaintext attack (KPA)**](https://en.wikipedia.org/wiki/Known-plaintext_attack) since:  

plaintext ⊕ ciphertext = key  

It's also trivial to flip arbitrary bits in the decrypted plaintext by manipulating the ciphertext. This is called 
[**malleability**](https://en.wikipedia.org/wiki/Malleability_(cryptography)).

## Known-plaintext attack explained

If we can guess, or we know, the initial plain text string (or at leas a part of it) and we know the result ciphertext we can guess the key 
used to encrypt. Once we get the key we can get the initial plain text string only reapplying the XOR function with the guessed key to the ciphertext:  

String ⊕ Key = Ciphertext → Ciphertext ⊕ Key = String

I wrotte a little pyhton script to show how this attack works and I added it to my collection of python examples.
You can take a look at the example here: [XorKpa.py](https://github.com/rubenhortas/python_examples/blob/master/Xor/XorKpa.py)

_Enjoy! ;)_