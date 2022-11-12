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

# How to disable history

To permanently disable the history we need to edit our ~/.bashrc file and add the next line:

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

_Enjoy! ;)_
