---
title: Hack the box BabyEncryption
date: 2022-05-01 00:00:01 +0000
categories: [hack the box, challenge]
tags: [hack the box, challenge, crypto, htb, babyencryption]
---

> You are after an organised crime group which is responsible for the illegal weapon market in your country. As a secret agent, you have infiltrated the group enough to be included in meetings with clients. During the last negotiation, you found one of the confidential messages for the customer. It contains crucial information about the delivery. Do you think you can decrypt it?

When we extract the BabyEncryption.zip file we found two files:
- msg.enc → Is the confidential message that you found during the last negotiation and you have to decrypt it.
- chall.py → The algorithm used for the organised crime group to encrypt their messages

We'll take a look at the chall.py script:

```python
import string
from secret import MSG

def encryption(msg):
    ct = []
    for char in msg:
        ct.append((123 * char + 18) % 256)
    return bytes(ct)

ct = encryption(MSG)
f = open('./msg.enc','w')
f.write(ct.hex())
f.close()
```

We can't import MSG because it's not a python standard library and we don't have the MSG library.
And we can see that, after encrypt the messages, the message is written in hexadecimal. So, we know that the string contained in msg.enc is in hexadecimal. Now we have to reverse the encryption.  

To get the message decrypted the first thing I did is revert the hexadecimal encoding converting the string content in msg.enc from hexadecimal to a bytes object.  

Then, since we have the function used to encrypt, I did a dictionary containing every ascii value and its encrypted value using the encryption function.  

Lastly, I went through the bytes object searching the unencrypted value of every encrypted character in the dictionary and concatenating the obtained values in a string that will be the flag.

To do this I did a self-explanatory python script that you can see here: [BabyDecryption.py]()


_Enjoy! ;)_
