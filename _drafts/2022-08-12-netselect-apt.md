---
title: netselect-apt
date: 2022-08-12 00:00:01 +0000
categories: [debian, administration]
tags: [gnu/linux , debian, administration]
---

netselect-apt is a script to create an apt sources.list file automatically by downloading the list of Debian mirrors and choosing the fastest.

## Install netselect-apt 

```shell
$ sudo apt install netselect-apt
```

## Run netselect-apt

We can choose our debian branch stable/testing/sid for the test

```shell
$ sudo netselect-apt -s stable
```

At this point, netselect-apt will have generated a sources.list file for the fastest mirror available on the directory where we have executed it.  
We can now substitute our /etc/apt/sources.list by the sources.list generated by netselect-apt or we can edit our /etc/apt/sources.list and sustitute de mirrors by the chosen one by netselect-apt.  
I always recommend to backup /etc/apt/sources.list before making changes.

_Enjoy! ;)_