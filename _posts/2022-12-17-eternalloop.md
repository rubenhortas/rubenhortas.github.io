---
title: Hack the box Eternal Loop
date: 2022-12-17 00:00:01 +0000
categories: [hack the box, challenge]
tags: [hack the box, challenge, misc, htb, Eternal loop, john, zip2john, rockyou, sqlite, sqlite3]
img_path: /assets/img/posts/
---

>Can you find a way out of this loop?

If we extract the Eternal Loop.zip file we found the 37366.zip file. This file is password protected and contains the 5900.zip file.  
We don't know the 37366.zip password... We'll try with 5900... It works!  
Now we know that the zip file will be the name of the inner file.  

Ok, the challenge now is unzip the files until we get the last file. We can do this by hand, but, **spoiler alert**: There will be 501 zips!  
To do this I did a self-explained python script that you can see here: [EternalLoop.py](https://github.com/rubenhortas/hackthebox/blob/main/eternalLoop/eternal_loop.py)

Once we have the last file, surprise, is password protected. 
This time we have not clue what the password might be, so we'll bruteforce it.

First of all is extract the hash of the zip file. We'll use zip2john

```console 
zip2john 6969.zip > 6969.hash
```

Once we have the hash we'll bruteforce it with john using the rockyou.txt file

```console
john --wordlist=/usr/share/wordlist/rockyou.txt 6969.hash
```

![bruteforcing with john](eternalloop_john.png)
_bruteforcing with john_ 

We get the password, so we can extract the 6969.zip file.

Once extracted we have a DoNotTouch file. The file is binary and, if we apply the file command to see what type of file it is, we can see that is a SQlite 3.x database.

![file command](eternalloop_file.png)
_file command_

So, we have to open the database

```console
sqlite3 DoNotTouch
```

Once the database is open we can list the tables with the command .tables 

```console
.tables
```

And we can export the tables data to csv files following the next steps:

- Configure to display headers
- Set the output mode

```console
.headers on
.mode csv
```

- And export the data for each table:

```console
.output data.csv
select * from table;
```

Now we can grep the csv files looking for the flag.  
To speed this step up I will tell you that the flag is in the employees table, in the email field for the employee wiht id 69.

```console
select Email from employees where EmployeeId = 69;
```

_Enjoy! ;)_
