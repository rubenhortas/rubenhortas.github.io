---
title: Hack the box LunaCrypt walkthrough
date: 2023-05-01 00:00:01 +0000
categories: [hack the box, walkthrough]
tags: [hack the box, challenge, crypto, htb, LunaCrypt]
---

>Our astronaut gained access to a key satellite and intercepted an encrypted message. The forensics team also recovered a file that looks like a custom encryption protocol. We're sure that these two elements are linked. Please can you help us reveal the contents of the secret message?

When we extract the LunaCrypt.zip file we found two files:  
- LunaCrypt.py → The script used for the encryption
- output.txt → The encrypted flag

We have to look at the code for decrypt the message.

# Analyzing LunaCrypt.py

## The output

We take a look at the function that writes the ouput:

```python
output = [f"{str(ord(v))} {str(ord(FLAGS[i]))}" for i, v in enumerate(CHARS)]
```

We can see that the output is written by pairs of numbers. The first number is an encrypted character (ct) and the second number is the flag used to encrypt it.

We take a look at the FLAG list generation:

```python
def AppendFlag(flag):
	FLAGS.append(strchr(bitxor(flag, 0x4A)))
```
Here we can see that the flags written in the output are xored with the key 0x4A.  
We need to keep this in mind in order to get the flags used for the characters encryption.

## The encryption

If we take a look to the function used to encrypt the characters:

```python
def EncryptCharacter(char):
	char = ValidateChar(char)
	flag = GenerateFlag()

	if CheckFlag(flag, FL_SWAPBYTES):
		char = ESwapChar(char)
	if CheckFlag(flag, FL_NEGATE):
		char = NegateChar(char)
	if CheckFlag(flag, FL_XORBY6B):
		char = XorBy6B(char)
	if CheckFlag(flag, FL_XORBY3E):
		char = XorBy3E(char)

	return char, flag
```

We can see that the first thing done is a character validation:

```python
strbyt = lambda x, y=0: ord(x[y])

def ValidateChar(char):
	if type(char) is str and len(char) == 1:
		char = strbyt(char)
	return char
```

The only thing that does the ValidateChar() function is return the ordinal of the character if the character has length 1, otherwise returns the character.

Then, the EncryptCharacter() function generates the flag. The flag generation is completely random and several flags may be activated at the same time.

Finally, the EncryptCharacter() function, checks if the flags FL_SWAPBYTES, FL_NEGATE, FL_XORBY6B and FL_XORBY3E are activated and, depending on which ones are activated, the encryption functions are applied to the character.

As several flags can be activated simultaneously, and the encrypting operations are applied in order, a character can undergo several encryptions.
We need to keep this in mind in order to reverse this part.

# Decrypting the message

## Reading the output

As the output is written by pairs of numbers, and the first number is the encrypted character and the second number is the encrypted flag we can read the output and separate the encrypted characters and the encrypted flags.

## Decrypting the flags

As the flags are encrypted xoring the flag with the 0x4A key, by the properties of XOR, we can obtain the plaintext flags only by xoring the encrypted flags with the same key.

```
pt ⊕ key = ct → ct ⊕ key = pt
```

## Reversing the EncryptCharacter() function

Since there can be several flags active at the same time and the encryption functions are applied in order, the only thing we need to do is to reverse the order of the functions applied:

```python
def DecryptCharacter(char, flag):
    char = ValidateChar(char)

    if CheckFlag(flag, FL_XORBY3E):
        char = XorBy3E(char)
    if CheckFlag(flag, FL_XORBY6B):
        char = XorBy6B(char)
    if CheckFlag(flag, FL_NEGATE):
        char = NegateChar(char)
    if CheckFlag(flag, FL_SWAPBYTES):
        char = InvertESwapChar(char)

    if type(char) is int:
        char = strchr(char)
    return char
```

## Reversing XorBy3E, XorBy6B and NegateChar

As these functions are only xorings (remember the XOR properties: pt ⊕ key = ct → ct ⊕ key = pt) and a negation, the inverse of these functions are the same functions.

## Reversing ESwapChar() function
This is the tricky part of the challenge.
Once you understand what it does, reversing it is pretty straightforward (although I spent more time on it than I'd like to assume...).

```python
def ESwapChar(char):
	char = ValidateChar(char)
	THIS_MSB = bitext(char, 4, 4)
	THIS_LSB = bitext(char, 0, 4)

	return strchr(bitbor(bitxor(THIS_MSB, 0x0D), bitxor(bitlst(THIS_LSB, 4), 0xB0)))
```

What does this function do?

The first thing this function does is validate the character (we've seen before that this returns the ordinal of the character if the character has length 1, otherwise returns the character).

Then declares two variables (in capital letters!? Really? ¬¬!): THIS_MSB and THIS_LSB

Lastly, this function applies some operations to THIS_MSB and THIS_LSB to get the encrypted char.

Let's simplify the encrypt function a bit:

```
* strchr = lambda x: chr(x)
* bitbor = lambda x, y: x | y → or
* bitxor = lambda x, y: x ^ y → xor
* bitlst = lambda x, y: x << y → Left shift, moves x to the left y places

  return strchr(bitbor(bitxor(THIS_MSB, 0x0D), bitxor(bitlst(THIS_LSB, 4), 0xB0))) →
→   ct = strchr(bitbor(bitxor(THIS_MSB, 0x0D), bitxor(bitlst(THIS_LSB, 4), 0xB0))) →
→   ct = chr(OR[XOR(THIS_MSB, 0x0D), XOR(bitlst(THIS_LSB, 4), 0xB0)])
→   ct = chr[(THIS_MSB ⊕ 0x0D) | (bitlst(THIS_LSB, 4) ⊕ 0xB0)]
```

We can see that the operation is a chr() applied over an OR between two XORs, so we'll need to start by reversing the two XORs.

### Left XOR: bitxor(THIS_MSB, 0x0D)

Ok, as this part it's a simpe XOR between THIS_MSB and 0x0D, what does THIS_MSB do?

```
* bitrst = lambda x, y: x >> y → Right shift, moves x to the right y places
* bitext = lambda x, y, z=1: bitrst(x, y) & int(math.pow(2, z) - 1) =  # the value of z will be overriden by the value of the caller function 
         = (x >> y) & int(math.pow(2, z) - 1)

* THIS_MSB = bitext(char, 4, 4) =
           = (char >> 4) & int(math.pow(2, 4) - 1) =  # SWAP! (char >> 4) converts the 4 most significant bits of the plaintext (pt) char into the 4 least significant bits of the encrypted char (ct)
           = (char >> 4) & int(16 - 1) =
           = (char >> 4) & 15
```

Let's see it in action:

```
pt = char = 'A' = 0100 **0001**

    0000 **0100** char >> 4 → SWAP!
AND 0000 1111 15 *
    ---------
    0000 0100 (char >>4) & 15 = THIS_MSB
XOR 0000 1101 0x0D
    ---------
    0000 1001 THIS_MSB ⊕ 0x0D = left XOR
```

_*As (0&0) = 0 and (0&1) = 1 and we are working only with the four least significant bits, this function does nothing and we can skip it to reverse this part_

### Right XOR: bitxor(bitlst(THIS_LSB, 4), 0xB0))

This part it's another simple XOR between bitlst(THIS_LSB, 4) and 0xB0. So, what does bitlst(THIS_LSB, 4) do?

```
* THIS_LSB = bitext(char, 0, 4)
* bitext = (x >> y) & int(math.pow(2, z) - 1)  # the value of z will be overriden by the value of the caller function 

→ THIS_LSB = (char >> 0) & int(math.pow(2, 4) - 1) =
           = char & int(math.pow(2, 4) - 1) = 
           = char & 15

* bitlst = lambda x, y: x << y  # SWAP! Left shift, moves x to the left y places

bitlst(THIS_LSB, 4) = bitext(char, 0, 4) =
                    = THIS_LSB << 4 =  #  SWAP! << 4 converts the 4 least significant bits of the plaintext char (pt) into the 4 most significant bits of the encrypted char (ct)
                    = (char & 15) << 4 
```

Let's see it in action:

```
pt = char = 'A' = 0100 0001

    0100 0001 char
AND 0000 1111 15 *
    ---------
    0000 **0001** char & 15 = THIS_LSB

    **0001** 0000 bitlst(THIS_LSB) = THIS_LSB << 4  # SWAP!
XOR 1011 0000 0xB0
    ---------
    1010 0000 bitlst(THIS_LSB) ⊕ 0xB0 = right XOR
```

_*As (0&0) = 0 and (0&1) = 1 and we are working only with the four least significant bits, this function does nothing and we can skip it to reverse this part_


### ct = chr(OR[XOR(THIS_MSB, 0x0D), XOR(bitlst(THIS_LSB, 4), 0xB0)])

```
ct = chr(OR[XOR(THIS_MSB, 0x0D), XOR(bitlst(THIS_LSB, 4), 0xB0)])
   = chr(OR[XOR(THIS_MSB, 0x0D), XOR(bitlst(THIS_LSB, 4), 0xB0)])
   = chr(OR(left XOR, right XOR) 
```

```
   0000 1001 left XOR
OR 1010 0000 right XOR
   ---------
   1010 1001 ct = ©
```

### Reversing the Left XOR: bitxor(THIS_MSB, 0x0D)

We part from ct = 1010 1001, and we know that the Left XOR give rise to the four least significant bits of the encrypted character (ct), so, first, we will clear the four most significant bits of the ct:
```
0000 1001 ct masked  # Four most significant bits cleared
```

Now, we need to reverse the XOR with 0x0D, in order to do this (and by the properties of XOR: pt ⊕ key = ct → ct ⊕ key = pt) we only need to XOR it with 0x0D again:  
```
    0000 1001 ct masked
XOR 0000 1101 0x0D
    ---------
	0000 0100 pt upper bits without swapping
```

Finally, we need to reverse the swapping to make the four least significant bits of the ct becomes again the four most significant bits of the pt:
```
0100 0000 pt upper bits without swapping << 4 = pt upper bits
```

### Reversing the Right XOR: bitxor(bitlst(THIS_LSB, 4), 0xB0))

We part from ct = 1010 1001, and we know that the Right XOR give rise to the four most significant bits of the encrypted character (ct), so, first, we will clear the four least significant bits of the ct:
```
1010 0000 ct masked (four least significant bits cleared)
```

Now, we need to reverse the XOR with 0xB0, in order to do this (and by the properties of XOR: pt ⊕ key = ct → ct ⊕ key = pt) we only need to XOR it with 0xB0 again:
```
    1010 0000 ct masked
XOR 1011 0000 0xB0
    ---------
	0001 0000 pt lower bits without swapping  # = bitlst(THIS_LSB, 4) = THIS_LSB << 4
```

Finally, we need to reverse the swapping to make the four most significant bits of the ct becomes again the four least significant bits of the pt:
```
0000 0001 pt lower bits without swapping >> 4 = pt lower bits
```

### Reversing the OR (getting the pt): chr(OR[XOR(THIS_MSB, 0x0D), XOR(bitlst(THIS_LSB, 4), 0xB0)])

To reverse the OR we only need to apply the OR again:
```
   0100 0000 pt upper bits
OR 0000 0001 pt lower bits
   ---------
   0100 0001 = pt = 'A'
```
And... We have our pt!

### The full InvertESwapChar() function:

```python
def InvertESwapChar(ct):
    # Left side -> Revert (THIS_MSB xor 0x0D)
    pt_upper = ValidateChar(ct)

    # Convert 4 upper bits to 0 (Masking pt_upper)
    for i in range(4, 8):
        pt_upper = pt_upper & ~(1 << i)

    pt_upper = pt_upper ^ 0x0D
    pt_upper = pt_upper << 4

    # Right side -> Revert (bitlst(THIS_LSB, 4) xor 0xB0)
    pt_lower = ValidateChar(ct)

    # Convert 4 lower bits to 0 (Masking pt_lower)
    for i in range(4):
        pt_lower = pt_lower & ~(1 << i)

    pt_lower = pt_lower ^ 0xB0  # = bitlst
    pt_lower = pt_lower >> 4  # bitlst >> 4

    pt = pt_upper | pt_lower

    return chr(pt)
```

To solve this challenge, I did a python script that you can see at: [luna_decrypt.py](https://github.com/rubenhortas/hackthebox/blob/main/lunaCrypt/luna_decrypt.py)

_Enjoy! ;)_

