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

# How to clear the history 

We can clear the current stored history with:

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

# Disable history permanently

To permanently disable the history we need to edit our ~/.bashrc file and set the HISTSIZE variable to 0:

```shell
HISTSIZE=0
```

Now, we need to reload our bash settings:

```shell
source ~/.bashrc
```

# Deleting the history

If we have disabled our history, we will delete our ~/.bash_history file to delete all the commands that have been registered:

```shell
rm ~/.bash_history
```

> Don't forget the history of the root account!
{: prompt-warning}

# Clear the history when closing the terminal

If we want to keep our terminal history and clear it when whe close the terminal, we can trap the sginal the signal that the terminal sends to the she sell.
To do this we need to edit our ~/.bashrc file and add the following line:

```shell
trap 'unset HISTFILE; exit' SIGHUP
```

# Clear the history on log out

If we want to delete our history when logging out, we can edit our ~/.bash_logout file and add the following command:

```shell
rm ~/.bash_history
```
> The ~/.bash_history contains commands that are executed when logging out.
{: prompt-info}

# Running a terminal without history

If we are using a GUI, we can run a free-history terminal, or something like that.
We can open a terminal, and once we have finished, we can send it a SIGKILL signal, avoiding to keep our commands logged in the history.

```shell
kill -9 $SHELL_PID
```

> Sending SIGKILL to our shell's PID will kill our shell without being able to do anything (such as save the history or execute ~/.bash_logout).
{: prompt-info}

# My personal choice

As we can see, there are a lot of aproximations to solve this problem.
Keep it for sure that there are more that aren't mentioned here... 
Some of them more complicated, others more refinated and others are meant for other purposes...

My personal choice, when I'm using bash, is a balance between usability and security, so I choose fully disable the root history and delete the user history on close the terminal (and the session if necessary).
This way, the root will never have history (and it's not a problem if we are using sudo) and the user will have a terminal and session history.
This means that the user (I mean we) will be able to repeat commands in a session using the arrow up, for example, thus facilitating its work.
This choice it's something like the zsh default behavior.

_Enjoy! ;)_
