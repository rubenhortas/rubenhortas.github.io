---
title: Disable bash history
date: 2022-11-15 00:00:01 +0000
categories: [privacy, bash]
tags: [privacy, bash, history]
---

Bash history is a log of the commands we have executed in bash.
By default it stores the last 1000 commands.
It stores our commands in the ~/.bash_history file.

There is situations on which we don't want to keep a record of our commands and we can delete and/or disable the bash history.

# How to delete history 

We can delete the current stored history with

```shell
history -c
```

# Do not record some commands

If we don't want to keep track of certain commands, for example ls and cd, we can edit our ~/.bashrc file and add the following line:

```
HISTIGNORE='ls*:cd*'
``` 

# Do not record some commands at will

We can omit any command lines which begin with a space in our history editing our ~/.bashrc file and adding the following line:

```
HISTCONTROL=ignorespace'
```

# How to permanently disable history

To permanently disable the history we need to edit our ~/.bashrc file and set the HISTSIZE variable to 0:

```
HISTSIZE=0
```

Or we can edit our ~/.bashrc file and add the next line:

```bash
set +o history
```

> set +o unsets the shell variable
{: prompt-info}

Now, we need to reload our bash settings:

```shell
source ~/.bashrc
```

Finally, we will delete our .bash_history file

```shell
rm ~/.bash_history
```

> Don't forget the history of the root account!
{: prompt-warning}

# When we can't edit ~/.bashrc nor ~/.bash_logout files: SIGKILL

Even though we can't edit ~/.bashrc nor ~/.bash_logout files, there's ways to avoid to keep our commands logged into history.  
Sending SIGKILL to our shell's PID will kill our shell without being able to do anythin (such as save the history).

```shell
kill -9 shell_pid
```

_Enjoy! ;)_
