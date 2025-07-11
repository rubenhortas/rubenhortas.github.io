---
title: Hack the box Deterministic
date: 2022-05-01 00:00:01 +0000
categories: [hack the box, challenge]
tags: [hack the box, challenge, misc, htb, Deterministic]
img_path: /assets/img/posts
---

## Challenge description

>There is a locked door in front of us that can only be opened with the secret passphrase. 
>There are no keys anywhere in the room, only this .txt. There is also a writing on the wall.. 
>"State 0: 69420, State N: 999, flag ends at state N, key length: one".. Can you figure it out and open the door?

## Solution

When we extract the `Deterministic.zip` file we only found the `deterministic.txt` file.

If we take a look at the `deterministic.txt` content, we can see a few things:

```
The states are correct but just for security reasons, 
each character of the password is XORed with a very super secret key.
100 H 110
110 T 111
111 B 112
112 { 113
113 l 114
114 0 115
115 l 116
116 _ 117
117 n 118
118 0 119
119 p 120
120 e 121
121 } 122
```

We can assume that the first column is the current state, the second column is the xored character of the password and the third column is the next state.
And the character of the password is **xored** with a very super secret **key with length one** (remember the statement) ;)

A deterministic system will always produce the same output from a initial state.
So, to solve this challenge, the first thing is get the secuence of states, starting from `69420` and ending in `999`.
Concatenating the characters associated to this states we will have the xored password.
Once we got the xored password, we need to bruteforce it with a key o length one (It shouldn't take long ;)) to get the plaintext password (our flag).
Lastly, once we have bruteforced the xored password, we will analyze the ouputs, and we will keep the only one that makes sense (and the only one that has the flag :P).

To solve this challenge I did a python script that you can see here: [deterministic.py](https://github.com/rubenhortas/hackthebox/blob/main/deterministic/deterministic.py)

*Enjoy! ;)*
