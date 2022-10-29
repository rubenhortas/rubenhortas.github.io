---
title: Find files containing specific text on GNU/Linux
date: 2022-06-24 00:00:01 +0000
categories: [howtos, grep]
tags: [gnu/linux, grep, howtos]
img_path: /assets/img/posts/
---

Sometimes we need to find all files containing a specific text withing the files, not in the file name.
We can achieve this with grep and a few parameters:

```shell
grep -irnwl /path -e 'pattern'
```

* -i ignore case
* -r recursive
* -n line number
* **-w match the whole word** (Do not use if not necessary)
* -l give the file name
* -e searched pattern

If we want to find several words we can do it using several -e options and arguments:

```shell
grep -irnwl /path -e 'pattern1' -e 'pattern2'
```

Or using the -E (extended regular expression) option and separating the words with the vertical bar:

```shell
grep -irnwE 'pattern1|pattern2' /path
```

_Enjoy! ;)_
