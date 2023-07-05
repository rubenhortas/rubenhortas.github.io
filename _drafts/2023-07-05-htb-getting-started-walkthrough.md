---
title: Hack the box Getting started walkthroug
date: 2023-07-05 00:00:01 +0000
categories: [hack the box, walkthrough]
tags: [hack the box, walkthrough, htb, getting started, getsimple, privilege escalation, privilege escalation, sudoers misconfiguration]
img_path: /assets/img/posts/
---

![getting started](htb-getting-started-desc.png)
*Getting started*


>In this article we are going to assume the folling ip addresses:
>
>Local machine (attacker): 10.0.0.1
>
>Target machine (victim): 10.0.0.2
{: .prompt-info}

This will be a black-box approach, because we don't have any information about the target.

# Enumeration

We are going to scan the tartget machine to find out what services are running:

```
# Nmap 7.94 scan initiated Tue Jul  4 13:26:04 2023 as: nmap -P0 -n -sV --open -oA gettingstarted_initial_scan 10.129.42.249
Nmap scan report for 10.129.42.249
Host is up (0.061s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue Jul  4 13:26:12 2023 -- 1 IP address (1 host up) scanned in 8.83 seconds
```

We can see that the target is a GNU/Linux (Ubuntu) host with a ssh server (OpenSSH 8.2p1) running on port 22 and a web service (Apache 2.4.41) running on port 80.

# Web footprinting

If we browse the target, we sill se a GetSimple welcome screen

![welcome screen](htb-getting-started-welcome-screen.png)
*Welcome screen*

The page doesn't seem to look to good.
If we take a look at the source code we can see a lot of references to `http://gettingstarted.htb`

![http://gettingstarted.htb references](htb-getting-started-getting-started-references.png)
*http://gettingstarted.htb references*

So we will add `10.0.0.2` as `gettingstarted.htb` to our `/etc/hosts`:

```
sudo echo `10.0.0.2 gettingstarted.htb` >> /etc/hosts
```

Now, the web, looks much better:

![welcome screen fixed](htb-getting-started-welcome-screen-fixed.png)
*Welcome screen fixed*

We can see that we have an instance of a **GetSimple** blog.
Now, we can start to identify the technologies in use.

## Technologies in use

```
$ whatweb gettingstarted.htb
http://gettingstarted.htb [200 OK] AddThis, Apache[2.4.41], Country[RESERVED][ZZ], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.41 (Ubuntu)], IP[10.129.42.249], Script[text/javascript], Title[Welcome to GetSimple! - gettingstarted]
```

## Directory enumeration

Let's see if there are hidden directories we can check to get more information...

```
$ gobuster dir -u http://gettingstarted.htb --wordlist /usr/share/dirb/wordlists/common.txt
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://gettingstarted.htb
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/dirb/wordlists/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Timeout:                 10s
===============================================================
XXXX/XX/XX 13:31:37 Starting gobuster in directory enumeration mode
===============================================================
/.hta                 (Status: 403) [Size: 283]
/.htaccess            (Status: 403) [Size: 283]
/.htpasswd            (Status: 403) [Size: 283]
/admin                (Status: 301) [Size: 324] [--> http://gettingstarted.htb/admin/]
/backups              (Status: 301) [Size: 326] [--> http://gettingstarted.htb/backups/]
/data                 (Status: 301) [Size: 323] [--> http://gettingstarted.htb/data/]
/index.php            (Status: 200) [Size: 5485]
/plugins              (Status: 301) [Size: 326] [--> http://gettingstarted.htb/plugins/]
/robots.txt           (Status: 200) [Size: 32]
/server-status        (Status: 403) [Size: 283]
/sitemap.xml          (Status: 200) [Size: 431]
/theme                (Status: 301) [Size: 324] [--> http://gettingstarted.htb/theme/]
```


*Enjoy! ;)*
