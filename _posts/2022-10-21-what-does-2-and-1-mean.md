---
title: What does "2>&1" mean?
date: 2022-10-21 00:00:01 +0000
categories: [unix, bash]
tags: [unix bash] 
---

Sometimes we met with shell commands that includes the "2>&1" string at the end.
If we don't the meaning of "2>&1" it's easy that we forgot the syntax.

So, what does "2>&1" mean?

2>&1 is the way to redirect the standard error (stderr) to the standard output (stdout).

How 2>&1 works?

The standard input (stdin) is the file descriptor 0.  
The standard output (stdout) is the file descriptor 1.  
The standard error (stderr) is the file descriptor 2.

">" is the unix file redirection operator. 
">" is used to redirect the contents of a command or file to another by overwritting it.

If we want to redirect the standard error (stderr, 2) to the standard output (stdout, 1) we could think of doing:

```shell
command 2>1
```

But this will redirect the standard error (stderr) to a file named 1.

"&" indicates that what follows is a file descriptor and not a file name, so the way to redirect the standard error (stderr) to the standard output (stdout) is:

```shell
command 2>&1
```

And, if we want to discard the exit of the command we can send it to the "bitbucket" (also known as /dev/null):

```shell
command > /dev/null  2>&1
```

/dev/null is a pseudo device (or virtual device) file in GNU/Linux. 
/dev/null works like a black hole, as soon as any data is written to /dev/null the data is deleted.

_Enjoy! ;)_
