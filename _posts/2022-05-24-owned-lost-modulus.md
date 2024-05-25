---
title: Hack the box Lost Modulus
date: 2022-05-24 00:00:01 +0000
categories: [hack the box, challenge]
tags: [hack the box, challenge, crypto, htb, Lost Modulus, RSA]
---

## Challenge description

> I encrypted a secret message with RSA but I lost the modulus. Can you help me recover it?

## Solution

When we extract the Lost Modulus.zip file we found two files:
- challenge.py → The script used for the custom RSA implementation
- output.txt → The encrypted flag

We take a look at the challenge.py script and the RSA implmentation seems legit...
We can see that the script is trying to open a flag.txt file:

```python
flag = open('flag.txt', 'r').read().strip().encode()
```

And we can see that the given encrypted flag is written in hexadecimal:

```python
print ('Flag:', crypto.encrypt(flag).hex())
``` 

Fist of all, I will apply the pep 8 coding conventios to the script, because it's giving me the creeps... :S

Then I will create the flag.txt file and with the hexadecimal flag contained in the output.txt file (without the 'Flag: ' part).

And now... I found two solutions for this challenge:

## The easy way

Since the script has the decrypt function, and seems legit, the first thing is to give it a try... Eventually it will works. So we can automatize this process with a little function:

```python
def decrypt_flag(crypto):
	encrypted_flag_hex = flag.decode()
	encrypted_flag_bytes = bytes.fromhex(encrypted_flag_hex)
	decrypted_flag_bytes = b'' 

	while b'HTB{' not in decrypted_flag_bytes: 
		decrypted_flag_bytes = crypto.decrypt(encrypted_flag_bytes)

	print('Decrypted flag bytes: ', decrypted_flag_bytes)

```

We call the function from the main passing crypto as argument and wait for it to finish.
To do this I only modified the challenge.py script. You can see the full script at [EasyWay.py](https://github.com/rubenhortas/hackthebox/blob/main/lostModulus/easy_way.py)

## The cube root attack

Is one of the simpliest attacks on RSA. 
When encrypting with low encryption exponents (i.e. e = 3) and small values of the m (i.e. m < n1/e), the result of m^e is strictly less than the modulus n. In this case, the modulus n loses its importance, has no effect, and the encryption goes from (m^e)%n to m^e; so: ct = (m^e)%n = m^e and ciphertext can be decrypted easily taking the eth root of the ciphertext: pt = sqrt(e, ct); in this case pt = sqrt(3, m).


To solve this challenge, I did a self-explanatory python script that you can see at [CubeRootAttack.py](https://github.com/rubenhortas/hackthebox/blob/main/lostModulus/cube_root_attack.py)

![Challenge results](owned-lost-modulus.png){: width="700" height="600" .shadow}
_Lost Modulus has been Pwned_


_Enjoy! ;)_
