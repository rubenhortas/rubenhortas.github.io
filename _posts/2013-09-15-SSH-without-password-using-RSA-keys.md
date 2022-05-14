---
title: SSH without password, using RSA keys
date: 2013-09-15 00:00:01 +0000
categories: [hardening, ssh]
tags: [hardening, ssh, rsa, network]
img_path: /assets/img/posts/
---
![Connection between computers](sshrsa.png){: width="280" }
_Connection between computers_

Suppose we have two computers: **computer1 with ip 10.0.0.1** and **computer2 with ip 10.0.0.2**.
We connect very frequently from **computer1** to **computer2** using SSH.
As we connect very frequently, entering the password every time we connect becomes a repetitive task and a waste of time.
In order to be able to connect between them without password, in a comfortable and secure way, we can do it using RSA keys instead.

To be able to connect via SSH using the RSA keys instead password:

We generate the public and private RSA keys on **computer1**:

```console
rubenhortas@computer1:~/.ssh$ ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/home/rubenhortas/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/rubenhortas/.ssh/id_rsa.
Your public key has been saved in /home/rubenhortas/.ssh/id_rsa.pub.
The key fingerprint is:
**:**:**:**:**:**:**:**:**:**:**:**:**:**:**:**
The key's randomart image is: 
```

_*You have to leave the passphrase empty, otherwise it will ask for it every time we connect to computer2 via SSH_

We copy the public key from **computer1** to **computer2**:

```console
rubenhortas@computer1$ ssh-copy-id -i ~/.ssh/id_rsa.pub 10.0.0.2
The authenticity of host '10.0.0.2 (10.0.0.2)' can't be established.
RSA key fingerprint is **:**:**:**:**:**:**:**:**:**:**:**:**:**:**:**.
Are you sure you want to continue connecting (yes/no)? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
rubenhortas@10.0.0.2's password:
```

Finally, we connect from **computer1** to **computer2** using SSH and checking that it does not ask us for the password.

_Enjoy! ;)_
