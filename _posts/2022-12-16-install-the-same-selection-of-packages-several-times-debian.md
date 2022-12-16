---
title: Install the same selection of packages several times on Debian
date: 2022-12-16 00:00:01 +0000
categories: [debian, administration]
tags: [debian, administration, apt, packages]
---

It can be useful to systematically install the same list of packages on several computers. 
Or keep a list of the installed packages in some computers, to expedite reinstallations.
In Debian this can be done quite easily.

First, we save the list of installed packages on the computer:

```shell
$ dpkg --get-selections > pkg-list
```

Next, If we want to install these packages in another computer,we transfer the pkg-list file onto the computers we want to update:

# Update dpkg's database of known packages.  
Record the list of available packages in the dpkg database:
```shell
# avail=`mktemp`
# apt-cache dumpavail > "$avail"
# dpkg --merge-avail "$avail"
# rm -f "$avail"
```

# Update dpkg's selections  
Restore the selection of packages that we wish to install:
```shell
# dpkg --set-selections < pkg-list
```

# Install the selected packages
```shell
# apt-get dselect-upgrade
```

> aptitude does not have this command! 
{: .prompt-warning}


_Source: [The Debian administrator's handbook - 6.2. aptitude, apt-get, and apt Commands](https://debian-handbook.info/browse/stable/sect.apt-get.html#sect.apt.install)_

_Enjoy! ;)_
