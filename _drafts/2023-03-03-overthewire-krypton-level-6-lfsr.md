---
title: Overthewire krypton level 6 LFSR
date: 2023-03-03 00:00:01 +0000
categories: [overthewire, krypton] 
tags: [overthewire, krypton, lfsr, cryptography]
---

Some time  ago I started to solve the [Krypton](https://overthewire.org/wargames/krypton/) game from 
[OverTheWire](https://overthewire.org/wargames/).

I did a [walkthrough about Kripton Level 4 and Krypton Level 5](https://rubenhortas.github.io/posts/overthewire-krytpon-levels-4-and-5-vigenere-cipher/).

At this point, the only level that I had to solve was the [Krypton Level 6](https://overthewire.org/wargames/krypton/krypton6.html).
This level consists in break a [stream cipher](https://en.wikipedia.org/wiki/Stream_cipher).
And, as the statement warns "*The challenge here is not simple*".

# Level Info

> Hopefully by now its obvious that encryption using repeating keys is a bad idea. Frequency analysis can destroy repeating/fixed key substitution 
crypto.  
>  
>A feature of good crypto is random ciphertext. A good cipher must not reveal any clues about the plaintext. Since natural language plaintext 
(in this case, English) contains patterns, it is left up to the encryption key or the encryption algorithm to add the ‘randomness’.  
>  
>Modern ciphers are similar to older plain substitution ciphers, but improve the ‘random’ nature of the key.  
>  
>An example of an older cipher using a complex, random, large key is a vigniere using a key of the same size of the plaintext. For example, 
imagine you and your confident have agreed on a key using the book ‘A Tale of Two Cities’ as your key, in 256 byte blocks.  
>  
>The cipher works as such:  
>  
>Each plaintext message is broken into 256 byte blocks. For each block of plaintext, a corresponding 256 byte block from the book is used 
as the key, starting from the first chapter, and progressing. No part of the book is ever re-used as key. The use of a key of the same length
 as the plaintext,
and only using it once is called a “One Time Pad”.  
>  
>Look in the krypton6 directory. You will find a file called ‘plain1’, a 256 byte block. You will also see a file ‘key1’, the first 256 bytes 
of ‘A Tale of Two Cities’.
>The file ‘cipher1’ is the cipher text of plain1. As you can see (and try) it is very difficult to break the cipher without the key knowledge.  
>  
>If the encryption is truly random letters, and only used once, then it is impossible to break. A truly random “One Time Pad” key cannot be broken.
>Consider intercepting a ciphertext message of 1000 bytes. One could brute force for the key, but due to the random key nature,
you would produce every single valid 1000 letter plaintext as well. Who is to know which is the real plaintext?!?  
>  
>Choosing keys that are the same size as the plaintext is impractical. Therefore, other methods must be used to obscure ciphertext 
against frequency analysis in a simple substitution cipher. 
The impracticality of an ‘infinite’ key means that the randomness, or entropy, of the encryption is introduced via the method.  
>  
>We have seen the method of ‘substitution’. Even in modern crypto, substitution is a valid technique. Another technique is ‘transposition’, 
or swapping of bytes.  
>  
>Modern ciphers break into two types; **symmetric and asymmetric**.  
>  
>**Symmetric ciphers come in two flavours: block and stream**.  
>  
>Until now, **we have been playing with classical ciphers, approximating ‘block’ ciphers**. A block cipher is done in **fixed size blocks** 
(suprise!). 
>For example, in the previous paragraphs we discussed breaking text and keys into 256 byte blocks, and working on those blocks.
>**Block ciphers use a fixed key to perform substituion and transposition ciphers on each block discretely**.  
>  
>**Its time to employ a stream cipher**. A stream cipher attempts to create an on-the-fly **‘random’ keystream to encrypt the incoming plaintext 
one byte at a time**.
>Typically, **the ‘random’ key byte is xor’d with the plaintext to produce the ciphertext**. **If the random keystream can be replicated 
at the recieving end,
then a further xor will produce the plaintext once again**.  
>  
>From this example forward, **we will be working with bytes**, not ASCII text, so a hex editor/dumper like hexdump is a necessity.
>Now is the right time to start to learn to use tools like cryptool.  
>  
>In this example, the keyfile is in your directory, however it is not readable by you. The binary ‘encrypt6’ is also available.
>It will read the keyfile and **encrypt any message you desire, using the key AND a ‘random’ number**.
>You get to **perform a ‘known ciphertext’ attack** by introducing plaintext of your choice.
>The challenge here is not simple, but **the ‘random’ number generator is weak**.  
>  
>As stated, it is now that we suggest you begin to use public tools, like cryptool, to help in your analysis.
>You will most likely need a hint to get going. See ‘HINT1’ if you need a kicktstart.  
>  
>If you have further difficulty, there is a hint in ‘HINT2’.  
>  
>The password for level 7 (krypton7) is encrypted with ‘encrypt6’.  
>  
>Good Luck!  

# What do we know so far about this level?

* It's a symmetric cipher
* It's a block cipher → Block ciphers use a fixed key to perform substituion and transposition ciphers on each block discretely
* It's a stream cipher where the "random" key byte (k) is xor'd with the plain text (pt) to produce the ciphertext (ct) → k ⊕ pt = ct
* k is a **"random"** keystream.
* If the random keystream (k) can be replicated at the recieving end, then a further xor will produce the plaintext once again 
→ k ⊕ pt = ct → pt ⊕ ct = k
* We will working with bytes.
* `encrypt6` will read the keyfile and encrypt any message using the **key** ***AND*** a **"*random*"** number.
* We have to perfom a [known ciphertext attack](https://en.wikipedia.org/wiki/Ciphertext-only_attack) 
* The random number generator is weak.
* The password for Krypton Level 7 is encrypted with ***encrypt6***.

# What can we learn from the clues?

## HINT1

> The 'random' generator has a limited number of bits, and is periodic.
Entropy analysis and a good look at the bytes in a hex editor will help.
  
There is a pattern!

## HINT2

> 8 bit LFSR

### [LFSR](https://en.wikipedia.org/wiki/Linear-feedback_shift_register)

[LFSR](https://en.wikipedia.org/wiki/Linear-feedback_shift_register) is used as a pseudo-random number generator for use in stream ciphers, 
but, as the register has a fine number of pssible states **it must eventually enter a repeating cycle**.
This means that **the numbers generated are periodic**. 

>A maximum-lenght LFSR will produce an m-secuence, so **a 8 bit LFSR will produce a 8 bit sequence**.
**The most commonly used linear function of single bits is exclusive-or (XOR ⊕)**.
>Thus, an [LFSR](https://en.wikipedia.org/wiki/Linear-feedback_shift_register) is most often a shift register whose input bit 
is driven by the XOR of some bits of the overall shift register value.
  
>Given a stretch of known plaintext and corresponding ciphertext, an attacker can intercept and recover a stretch of LFSR output stream 
used in the system described, and from that stretch of the output stream can construct an LFSR of minimal size that simulates 
the intended receiver by using the Berlekamp-Massey algorithm. This LFSR can then be fed the intercepted stretch of output stream 
to recover the remaining plaintext.

[LFSR](https://en.wikipedia.org/wiki/Linear-feedback_shift_register) works by feeding on their own output.
They are constructed in a way that makes them endlessly cycle through a pattern of values while outputting that seemingly random pattern.

There's a very graphic and good explanation about how a [LFSR](https://en.wikipedia.org/wiki/Linear-feedback_shift_register) works 
in [Random Numbers with LFSR (Linear Feedback Shift Register) - Computerphile](https://www.youtube.com/watch?v=Ks1pw1X22y4).

# The journey to solution

## Fail 1

From the properties of XOR we know that:

```
plaintext ⊕ key = ciphertext → ciphertext ⊕ key = plaintext
```

And we know that:

```
p ⊕ 0 = p
```

Because:

| p | 0 | p⊕0 |
|---|---|-----|
| 0 | 0 | 0   |
| 1 | 0 | 1   |

So, If we put a bunch of zeros as entry, we'll get the key directly:

```
plaintext ⊕ key = ciphertext →
        0 ⊕ key = ciphertext →
        0 ⊕ key = key → key  = ciphertext
```

```shell
krypton6@bandit:/tmp/rubenhortas$ ln -s /krypton/krypton6/keyfile.dat 
krypton6@bandit:/tmp/rubenhortas$ python3 -c "print('0'*50)" > zeros_pt.txt
krypton6@bandit:/tmp/rubenhortas$ /krypton/krypton6/encrypt6 zeros_pt.txt zeros_ct.txt
```

And our key is... An empty file! **FAIL!**

## Fail 2

Ok, encrypt6 doesn't seem to work with numbers, so let's try with letters:

```shell
krypton6@bandit:/tmp/rubenhortas$ python3 -c "print('A'*50)" > as_pt.txt
krypton6@bandit:/tmp/rubenhortas$ /krypton/krypton6/encrypt6 as_pt.txt as_ct.txt
krypton6@bandit:/tmp/rubenhortas$ cat as_pt.txt 
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
krypton6@bandit:/tmp/rubenhortas$ cat as_ct.txt 
EICTDGYIYZKTHNSIRFXYCPFUEOCKRNEICTDGYIYZKTHNSIRFXY
```

```shell
krypton6@bandit:/tmp/rubenhortas$ python3 -c "print('B'*50)" > bs_pt.txt
krypton6@bandit:/tmp/rubenhortas$ /krypton/krypton6/encrypt6 bs_pt.txt bs_ct.txt
krypton6@bandit:/tmp/rubenhortas$ cat bs_pt.txt 
BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB
krypton6@bandit:/tmp/rubenhortas$ cat bs_ct.txt 
FJDUEHZJZALUIOTJSGYZDQGVFPDLSOFJDUEHZJZALUIOTJSGYZ
```

```shell
krypton6@bandit:/tmp/rubenhortas$ python3 -c "print('C'*50)" > /tmp/rubenhortas/cs_pt.txt
krypton6@bandit:/tmp/rubenhortas$ /krypton/krypton6/encrypt6 cs_pt.txt cs_ct.txt
krypton6@bandit:/tmp/rubenhortas$ cat cs_pt.txt 
CCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCC
krypton6@bandit:/tmp/rubenhortas$ cat cs_ct.txt 
GKEVFIAKABMVJPUKTHZAERHWGQEMTPGKEVFIAKABMVJPUKTHZA
```

We can see some curious things:

* If we encrypt the same input (or plaintext) many times we'll see the same output (or ciphertext) → We can forget about the random part 
for the key

* The ciphertext will be repeated from the 30th character → The key will have a max length of 30 characters  
   For example, the `as_ciphertext.txt`:  
   **EICTDGYIYZKTHNSIRFXYCPFUEOCKRN**EICTDGYIYZKTHNSIRFXY →  
   → EICTDGYIYZKTHNSIRFXYCPFUEOCKRN  
   → EICTDGYIYZKTHNSIRFXY  
   
   So, we can short our plaintexts and ciphertext to 30, to make our work easier:
   
   | Plaintext                      | Ciphertext                     |
   |--------------------------------|--------------------------------|
   | AAAAAAAAAAAAAAAAAAAAAAAAAAAAAA | EICTDGYIYZKTHNSIRFXYCPFUEOCKRN |
   | BBBBBBBBBBBBBBBBBBBBBBBBBBBBBB | FJDUEHZJZALUIOTJSGYZDQGVFPDLSO |
   | CCCCCCCCCCCCCCCCCCCCCCCCCCCCCC | GKEVFIAKABMVJPUKTHZAERHWGQEMTP |
   
* The ciphertext has the same length than the plaintext → Our ciphertext has 15 characters, so we need, at least, a 15 characters key.

The first thing is to get the keys for our three ciphers and compare if they are the same.
We'll get the keys xoring the plaintext with the ciphertext, because: 

```shell
plaintext ⊕ key = ciphertext → key = plaintext ⊕ ciphertext
```

The keys:

```shell
as_key: b'\x04\x08\x02\x15\x05\x06\x18\x08\x18\x1b\n\x15\t\x0f\x12\x08\x13\x07\x19\x18\x02\x11\x07\x14\x04\x0e\x02\n\x13\x0f'
bs_key: b'\x04\x08\x06\x17\x07\n\x18\x08\x18\x03\x0e\x17\x0b\r\x16\x08\x11\x05\x1b\x18\x06\x13\x05\x14\x04\x12\x06\x0e\x11\r'
cs_key: b'\x04\x08\x06\x15\x05\n\x02\x08\x02\x01\x0e\x15\t\x13\x16\x08\x17\x0b\x19\x02\x06\x11\x0b\x14\x04\x12\x06\x0e\x17\x13'
```

Ok, the keys are different... **FAIL 2!**

# Solution

So now, we know that the function is deterministic, it always will give the same output (or ciphertext) for a given input (or plaintext), 
and the key will depend on the input (or the plaintext).

If we take a closer look to the plaintexts and the ciphertexts we can see that when we increment by one character our input (or plaintext),
the output (or ciphertext) will also be increased by one character.

| Plaintext                      | Ciphertext                     |
|--------------------------------|--------------------------------|
| AAAAAAAAAAAAAAAAAAAAAAAAAAAAAA | EICTDGYIYZKTHNSIRFXYCPFUEOCKRN |
| BBBBBBBBBBBBBBBBBBBBBBBBBBBBBB | FJDUEHZJZALUIOTJSGYZDQGVFPDLSO |
| CCCCCCCCCCCCCCCCCCCCCCCCCCCCCC | GKEVFIAKABMVJPUKTHZAERHWGQEMTP |
   
So, we can asume that the difference between the plaintext and the ciphertext will be constant.

I think, at this point we can solve this challenge with [linear cryptanalysis](https://en.wikipedia.org/wiki/Linear_cryptanalysis), 
[differential cryptanalisys](https://en.wikipedia.org/wiki/Differential_cryptanalysis) 
or [Berlekamp–Massey algorithm](https://en.wikipedia.org/wiki/Berlekamp%E2%80%93Massey_algorithm)...
I don't know for sure.
I was researching (and reading algebraic theories beyond my means) when I did another test that gave me a clue to find a easier solution:

```shell
krypton6@bandit:/tmp/rubenhortas$ echo "ABC" > abc_pt.txt
krypton6@bandit:/tmp/rubenhortas$ /krypton/krypton6/encrypt6 abc_pt.txt abc_ct.txt
krypton6@bandit:/tmp/rubenhortas$ cat abc_pt.txt 
ABC
krypton6@bandit:/tmp/rubenhortas$ cat abc_ct.txt 
EJE
```

We can see that when encripting a plaintext character in a certain position, it always give the same exact encrypted character in the same 
exact position...

| Plaintext | Ciphertext |
|-----------|------------|
| A         | **(E)**    |
| B**(B)**  | F**(J)**   |
| CC**(C)** | GK**(E)**  |
| ABC       | EJE        |

So I did a 26x30 matrix encrypting every english alphabet character 30 times (the length of the LFSR period).
And the plaintext password for the Krypton Level 7 will be the result of find the plaintext character corresponding to the encrypted character 
in the corresponding position in our matrix:

|       |   P   |   N   |   U   |   K   |   L   |   Y   |   L   |   W   |   R   |   Q   |   K   |   G   |   K   |   B   |   E   |      ...      |
|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|---------------|
|**(A)**|   E   |   I   |   C   |   T   |   D   |   G   |   Y   |   I   |   Y   |   Z   |**(K)**|   T   |   H   |   N   |   S   |IRFXYCPFUEOCKRN|
|   B   |   F   |   J   |   D   |   U   |   E   |   H   |   Z   |   J   |   Z   |   A   |   L   |   U   |   I   |   O   |   T   |JSGYZDQGVFPDLSO|
|   C   |   G   |   K   |   E   |   V   |   F   |   I   |   A   |   K   |   A   |   B   |   M   |   V   |   J   |   P   |   U   |KTHZAERHWGQEMTP|
|**(D)**|   H   |   L   |   F   |   W   |   G   |   J   |   B   |   L   |   B   |   C   |   N   |   W   |**(K)**|   Q   |   V   |LUIABFSIXHRFNUQ|
|   E   |   I   |   M   |   G   |   X   |   H   |   K   |   C   |   M   |   C   |   D   |   O   |   X   |   L   |   R   |   W   |MVJBCGTJYISGOVR|
|**(F)**|   J   |**(N)**|   H   |   Y   |   I   |   L   |   D   |   N   |   D   |   E   |   P   |   Y   |   M   |   S   |   X   |NWKCDHUKZJTHPWS|
|   G   |   K   |   O   |   I   |   Z   |   J   |   M   |   E   |   O   |   E   |   F   |   Q   |   Z   |   N   |   T   |   Y   |OXLDEIVLAKUIQXT|
|   H   |   L   |   P   |   J   |   A   |   K   |   N   |   F   |   P   |   F   |   G   |   R   |   A   |   O   |   U   |   Z   |PYMEFJWMBLVJRYU|
|**(I)**|   M   |   Q   |   K   |   B   |**(L)**|   O   |   G   |   Q   |   G   |   H   |   S   |   B   |   P   |   V   |   A   |QZNFGKXNCMWKSZV|
|   J   |   N   |   R   |   L   |   C   |   M   |   P   |   H   |   R   |   H   |   I   |   T   |   C   |   Q   |   W   |   B   |RAOGHLYODNXLTAW|
|   K   |   O   |   S   |   M   |   D   |   N   |   Q   |   I   |   S   |   I   |   J   |   U   |   D   |   R   |   X   |   C   |SBPHIMZPEOYMUBX|
|**(L)**|**(P)**|   T   |   N   |   E   |   O   |   R   |   J   |   T   |   J   |   K   |   V   |   E   |   S   |   Y   |   D   |TCQIJNAQFPZNVCY|
|**(M)**|   Q   |   U   |   O   |   F   |   P   |   S   |   K   |   U   |   K   |   L   |   W   |   F   |   T   |   Z   |**(E)**|UDRJKOBRGQAOWDZ|
|**(N)**|   R   |   V   |   P   |   G   |   Q   |   T   |**(L)**|   V   |   L   |   M   |   X   |**(G)**|   U   |   A   |   F   |VESKLPCSHRBPXEA|
|**(O)**|   S   |   W   |   Q   |   H   |   R   |   U   |   M   |**(W)**|   M   |   N   |   Y   |   H   |   V   |**(B)**|   G   |WFTLMQDTISCQYFB|
|   P   |   T   |   X   |   R   |   I   |   S   |   V   |   N   |   X   |   N   |   O   |   Z   |   I   |   W   |   C   |   H   |XGUMNREUJTDRZGC|
|   Q   |   U   |   Y   |   S   |   J   |   T   |   W   |   O   |   Y   |   O   |   P   |   A   |   J   |   X   |   D   |   I   |YHVNOSFVKUESAHD|
|**(R)**|   V   |   Z   |   T   |**(K)**|   U   |   X   |   P   |   Z   |   P   |**(Q)**|   B   |   K   |   Y   |   E   |   J   |ZIWOPTGWLVFTBIE|
|**(S)**|   W   |   A   |**(U)**|   L   |   V   |**(Y)**|   Q   |   A   |   Q   |   R   |   C   |   L   |   Z   |   F   |   K   |AJXPQUHXMWGUCJF|
|**(T)**|   X   |   B   |   V   |   M   |   W   |   Z   |   R   |   B   |**(R)**|   S   |   D   |   M   |   A   |   G   |   L   |BKYQRVIYNXHVDKG|
|   U   |   Y   |   C   |   W   |   N   |   X   |   A   |   S   |   C   |   S   |   T   |   E   |   N   |   B   |   H   |   M   |CLZRSWJZOYIWELH|
|   V   |   Z   |   D   |   X   |   O   |   Y   |   B   |   T   |   D   |   T   |   U   |   F   |   O   |   C   |   I   |   N   |DMASTXKAPZJXFMI|
|   W   |   A   |   E   |   Y   |   P   |   Z   |   C   |   U   |   E   |   U   |   V   |   G   |   P   |   D   |   J   |   O   |ENBTUYLBQAKYGNJ|
|   X   |   B   |   F   |   Z   |   Q   |   A   |   D   |   V   |   F   |   V   |   W   |   H   |   Q   |   E   |   K   |   P   |FOCUVZMCRBLZHOK|
|   Y   |   C   |   G   |   A   |   R   |   B   |   E   |   W   |   G   |   W   |   X   |   I   |   R   |   F   |   L   |   Q   |GPDVWANDSCMAIPL|
|   Z   |   D   |   H   |   B   |   S   |   C   |   F   |   X   |   H   |   X   |   Y   |   J   |   S   |   G   |   M   |   R   |HQEWXBOETDNBJQM|
 
And we got our password for the Krypton Level 7!

*Enjoy! ;)*