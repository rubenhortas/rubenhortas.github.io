---
title: Process argument spoofing in GNU/Linux
date: 2024-07-20 00:00:01 +0000
categories: [programming, argument spoofing]
tags: [programming, spoofing]
---

Process argument spoofing (command line spoofing or program argument spoofing) is a technique that allow us to spawn a process with arguments and override those arguments at execution time.  
With this technique we can hide and/or modify the real arguments of the process.  
So, we can do things as change the program name, hide the program arguments or show other arguments.  

The arguemnts are only positions in memory, so we can overwrite the process arguments by overwritting the process memory.  
Once we have overwritten the memory, the results given by monitoring tools are altered and will show our fake arguments.

## Why and/or when would we want to do this?  

We may want run a program with a password, a user name and/or a path as arguments and we don't want them to be visible.  
Maybe we are doing a CTF challenge (or a pentest) and we are in a scenario when we could use it, together with other techniques, to escalate privileges.
Maybe we want to run a quickly (and poorly) written program, or, maybe, we are trying to create a new malware and hide it.   
We'll talk about this later.

## POC full code

First of all, I did a gist of the POC (Proof Of Concept) where you can see the following code blocks in context.  
You can see the POC full code here: [process_argument_spoofer_linux.c](https://gist.github.com/rubenhortas/1bfe50673297c975d979060e0af97d49)

## Hidding the arguments

We can hide the program arguments overwriting them with nulls (or whitespaces):

```C
len = strlen(argv[i]);
memset(argv[i], 0, len); // Overwrite everything with null
```

And, now, we will overwrite the program name, it's not mandatory but it will be fun.  
We can overwite it with anything, but what if we overwrite it with sometihng like a kernel process?  
This way our program will have a better chance of going unnoticed.

```C
strcpy(argv0, "[kworker fake/1:1-events]"); // Overwrite the program name
```

Let's compile and run it:

```shell
gcc -O3 process_argument_spoofer_linux.c -o spoofer.out && ./spoofer.out -p myverysecretpassword
PID: 20679
argv[0] './spoofer.out', address '0x7ffe3c5a22d8'
argv[1] '-p', address '0x7ffe3c5a2afc'
argv[2] 'myverysecretpassword', address '0x7ffe3c5a2aff'
```

Let's see what `top` says:

```shell
ps aux | grep -i 20679 | head -n1
rubenhortas      20679 99.0  0.0   2460   852 pts/2    R+   00:01   0:16 [kworker fake/1:1-events]
```

Ok, so now we have our program disguised as a kernel proceess (or almost :P).

## Spoofing the arguments

We could have kept the program name and overwritten the arguments to get something like:

```shell
rubenhortas      20679 99.0  0.0   2460   852 pts/2    R+   00:01   0:16 /spoofer.out -p ********************
```

Also, we could have kept the program name and hidden the arguments:

```shell
rubenhortas      20679 99.0  0.0   2460   852 pts/2    R+   00:01   0:16 /spoofer.out
```

We can do what we want, all we have to do is modify the code.

## Why is it a bad idea?

Hiding (or spoofing) the arguments this way it's fine in certain scenarios, as when we are doing a CTF challenge, but, if your idea is to use it to do a malware, I have bad news for you: this technique is easily discovered by malware detecting tools (sensors, monitors, etc). 
Also, **program argument spoofing will should never be used in a production environment**, because it's trivial to unmask it using tools as [pspy](https://github.com/DominicBreuker/pspy):

```shell
2024/07/20 00:01:00 CMD: UID=1000  PID=20697  | ./spoofer.out -p myverysecretpassword
```

Pwned :(

*Enjoy! :)*
