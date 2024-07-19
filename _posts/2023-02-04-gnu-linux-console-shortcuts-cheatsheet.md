---
title: GNU/Linux console shortcuts cheatsheet
date: 2023-02-04 00:00:01 +0000
categories: [gnu/linux, cheatsheets]
tags: [gnu/linux, console, cheatsheet]
---

Some useful shortcuts to speed up our console use.

# Editing

* `[TAB]` or `[CTRL] + i` Autocomplete.

* `[CTRL] + u` Delete everything from the cursor position to the beginning of the line.
* `[CTRL] + k` Delete everything from the cursor position to the end of the line.
* `[CTRL] + w` Delete the word before the cursor.
* `[CTRL] + d` Delete the character following the cursor.
* `[CTRL] + h` Delete the character before the cursor.
* `[CTRL] + SHIFT + c` Copy.
* `[CTRL] + SHIFT + v` Paste.

* `[ALT] + d` Delete until the end of the word.
* `[ALT] + [BACKSPACE]` Delete until the start of the word.
* `[ALT] + t` Switch current word with previous word.
* `[ALT] + l/u` Change the next word to lowercase/uppercase.
* `[ALT] + .` Show the last word of the last used command.

# History

* `[CTRL] + r` Search through command history for commands that match our search patterns.
* `[↑]` or `[CTRL] + p` Previous command.
* `[↓]` or `[CTRL] + n` Next command.

# Navigation

* `[CTRL] + a` Move the cursor to the beginning of the current line.
* `[CTRL] + e` Move the cursor to the end of the current line.
* `[CTRL] + b/f` Jump backward/forward one character.
* `[CTRL] + [←]/[→]` Jump at the beginning of the current/previous word.

* `[ALT] + b/f` Jump backward/forward one word.

# Tasks and processes

* `[CTRL] + s` Stop command output to the screen.
* `[CTRL] + c` Sends SIGINT signal to the current task/process.
* `[CTRL] + d` Close STDIN pipe
* `[CTRL] + z` Sends the SIGTSTP signal to the current process. Supend current command and send it to background.
* `[CTRL] + q` Resume suspended command.

# Misc

* `[CTRL] + l` Clear the terminal.
* `[CTRL] + [+]/[-]` Zoom in/out.

> There are many more shortcuts (and they are console dependent). I only put the most useful for me.
{: .prompt-info}

*Enjoy! ;)*
