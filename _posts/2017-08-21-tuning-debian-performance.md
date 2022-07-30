---
title: Tuning debian performance
date: 2017-08-21 00:00:01 +0000
categories: [debian, performance]
tags: [debian, tuning, performance, gnu/linux]
---

For personal reasons I love tuning the performance of my operating systems.
My personal choice is, usually, [Debian GNU/Linux](https://www.debian.org/).
I always part from installing a base system from a netinstall image and then I uninstall some packages that I won't need.
Lastly I change a few things to optmize performance and have a lighter gnu/linux operating system:

- Don't install any display manager (kdm, gdm, xdm, ldm...)
- Install a windows manager instead a desktop environment (my personal prefference now is awesomewm)
- Reduce swap consumption or disable swap  
	We can edit the swappiness file that controls the trend kernel to swapping (default is 60)
	(This is my choice for systems where I have swap partitions: desktop, laptops...)

	```console
	# echo 10 > /proc/sys/vm/swappiness
	```

	We can disable the swapping with **swapoff**  
	
	```console
	# swapoff -a
	```

- Disable unused services  
	If we are using systemd we can disable services on boot by running:

	```console
	# systemctl disable service-name
	```

	We also can use the text-based console tool **sysv-rc-conf** to disable (or enable) services for each runlevel
	(remember that debian does not make any difference between runlevels 2-5).

_Enjoy! ;)_
