---
title: hack the box Nibbles walkthrough 
date: 2023-07-09 00:00:01 +0000
categories: [hack the box, walkthrough]
tags: [hack the box, walkthrough, htb, nibbles, nibbleblog, privilege escalation, world-writable file, sudoers missconfiguration, eJPT]
img_path: /assets/img/posts
---

![nibbles](htb-nibles-desc.png)
_nibbles_

As this is a retired machine, this will be a grey-box approach, because we have some information about the target.

Things we know about the target machine:

* OS: **Linux**
* User path: **Web**
* Privilege escalation: **World-writable File / Sudoers Misconfiguration**

From this information we can assume that the target machine will be a GNU/Linux host with a running web server with two attack vectors (a world writable file and a misconfigured sudo).

# Enumeration

We are going to scan the target machine to see if our suspicions are confirmed.

```
nmap -P0 -n -sV --open -oA nibbles_inital_scan 10.0.0.2
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
_/nibles_directory_

## Technologies in use

We can identify the technologies used with `whatweb`

```
http://10.0.0.2/nibbleblog [301 Moved Permanently] Apache[2.4.18], Country[RESERVED][ZZ], HTTPServer[Ubuntu Linux][Apache/2.4.18 (Ubuntu)], IP[10.0.0.2], RedirectLocation[http://10.0.0.2/nibbleblog/], Title[301 Moved Permanently]
http://10.0.0.2/nibbleblog/ [200 OK] Apache[2.4.18], Cookies[PHPSESSID], Country[RESERVED][ZZ], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.18 (Ubuntu)], IP[10.0.0.2], JQuery, MetaGenerator[Nibbleblog], PoweredBy[Nibbleblog], Script, Title[Nibbles - Yum yum]

```

## Directory enumeration

As `/nibbleblog` is hidden, let's see if there are more directories hidden.
For this we are going to use `gobuster`:

```
gobuster dir -u http://10.0.0.2/nibbleblog/ --wordlist /usr/share/dirb/wordlists/common.txt
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

We can see that there is an `admin.php` page...

![admin.php](htb-nibbles-nibbles-admin.png)
_admin.php_

Unfortunately we don't have valid credentials, and the blog has bruteforcing protection.

If we browse the other paths, we can found a `users.xml` file in `/nibleblog/content/private/`.
Here, we can found our valid admin user:

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

# Foothold

# Privilege escalation


*Enjoy! ;)*
