---
title: Delete and disable bash history
date: 2022-12-02 00:00:01 +0000
categories: [privacy, bash]
tags: [privacy, bash, history]
---

Bash history is a log of the commands we have executed in bash.
By default it stores the last 1000 commands.
It stores our commands in the ~/.bash_history file.

There is situations on which we don't want to keep a record of our commands and we can delete and/or disable the bash history.

# How to delete history 

We can delete the current stored history with:

```shell
history -c
```

# Ignore some commands

If we don't want to keep track of certain commands, for example ls and cd, we can edit our ~/.bashrc file and add the following line:

```shell
HISTIGNORE='ls*:cd*'
```

# Ignore some commands at will

We can omit record any command lines which begin with a space in our history editing our ~/.bashrc file and adding the following line:

```
HISTCONTROL=ignorespace'
```

# Disable the history by unsetting the history shell variable

We can disable our history for our current session by unsetting the history shell variable executing the following command:

```shell
set +o history
```

We can execute this command in a terminal or we can append it to our ~/.bashrc file.

> Use this method is a good solution when we can't edit our ~/.bashrc file
{: prompt-info}

# Permanently disable history

To permanently disable the history we need to edit our ~/.bashrc file and set the HISTSIZE variable to 0:

```shell
HISTSIZE=0
```

Now, we need to reload our bash settings:

```shell
source ~/.bashrc
```

Finally, we will delete our ~/.bash_history file

```shell
rm ~/.bash_history
```

> Don't forget the history of the root account!
{: prompt-warning}

# Running a terminal without history

If we are using a GUI, we can run a free-history terminal, or something like that.
We can open a terminal, and once we have finished, we can send it a SIGKILL signal, avoiding to keep our commands logged in the history.  

```shell
kill -9 $SHELL_PID
```

> Sending SIGKILL to our shell's PID will kill our shell without being able to do anything (such as save the history).
{: prompt-info}


_Enjoy! ;)_
