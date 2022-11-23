---
title: Delete and disable bash history
date: 2022-12-02 00:00:01 +0000
categories: [privacy, bash]
tags: [privacy, bash, history]
---

Bash history is a log of the commands we have executed in bash.
By default it stores the last 1000 commands in the ~/.bash_history file.

There is situations on which we don't want to keep a record of our commands, and we can avoid, delete and/or disable the bash history.

# How to clear the history 

We can clear the current stored history with:

```shell
history -c
```

# Having permissions to edit ~/.bashrc and/or ~/.bash_logout

## Ignore some commands

If we don't want to keep track of certain commands, for example "ls" and "cd", we can edit our ~/.bashrc file and add the following line:

```
HISTIGNORE='ls*:cd*'
```

## Ignore some commands at will

We can omit record any command lines which begin with a space in our history editing our ~/.bashrc file and adding the "ignorespace" option to the HISTCONTROL variable:

```
HISTCONTROL='ignorespace'
```

>If HISTCONTROL value is "ignoreboth" means that "ignorespace" and "ignoredups" (ignore duplicated entries) are enabled.
{: prompt-info}

## Disabling the history permanently

To permanently disable the history we need to edit our ~/.bashrc file and set the HISTSIZE variable to 0:

```
HISTSIZE=0
```

Now, we need to reload our bash settings:

```shell
source ~/.bashrc
```

## Clearing the history when closing the terminal

If we want to keep our terminal history and clear it when whe close the terminal, we can trap the sginal that the terminal sends to the shell.
To do this we need to edit our ~/.bashrc file and add the following line at the end:

```shell
trap 'unset HISTFILE; exit' EXIT
```

## Clear the history on log out

If we want to delete our history when logging out, we can edit our ~/.bash_logout file and add the following command:

```shell
rm ~/.bash_history
```

> The ~/.bash_logout contains commands that are executed when logging out.
{: prompt-info}

## Deleting the history

If we have disabled our history, we will delete our ~/.bash_history file to delete all the commands that have been registered until now:

```shell
rm ~/.bash_history
```

> Don't forget the history for the root account!
{: prompt-warning}

# Without having permissions to edit ~/.bashrc

## Disablig the history by unsetting the history shell variable

We can disable our history for our current session by unsetting the history shell variable executing the following command:

```shell
set +o history
```

> We could also append this command to our ~/.bashrc file, but don't have much sense since, if we can edit our ~/.bashrc we can set the HISTSIZE to 0.
{: prompt-info}

## SIGKILL

Sending SIGKILL to our shell's PID will kill our shell without being able to do anything (such as save the history or execute ~/.bash_logout).

```shell
kill -9 $$
```

>$$ expands to the PID of the shell process executing the command
{: prompt-info}

# My personal choice

As we can see, there are a lot of ways to solve this problem.
Keep it for sure that there are more that aren't mentioned here... 
Some of them are variations, others are more easy or more complicated, others more refinated and others are meant for other purposes...

My personal choice, when I'm using bash, it's what I believe a balance between usability and security. So I choose fully disable the root history, keeping a (relativelly) low HISTSIZE for the user and deleting the user history when closing the terminal (and the session if necessary).
This way, the root will never have history (it's not a problem if we use sudo) and the user will have a temporary terminal and/or session history.
This means that the user (I mean we) will be able to repeat commands in a session using the history (or the up arrow key) facilitating its work.
This choice it's something like the zsh default behavior.

_Enjoy! ;)_
