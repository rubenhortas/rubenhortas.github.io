---
title: Tuning debian performance
date: 2017-08-21 00:00:01 +0000
categories: [debian, performance]
tags: [debian, tuning. performance, gnu/linux]
img_path: /assets/img/posts/
---

For personal reasons I love tuning the performance of my operating systems.
I always from installing a base system from a netinstall image and then I uninstall some packages that I won't need.
And then I change a few things to optmize performance and have a lighter operating system.

- Do not install any display manager (kdm, gdm, xdm, ldm...)
- Install a windows manager instead a desktop environment (my personal prefference no is awesomewm)
- Reduce swap consumption or disable swap
	We can edit the swappiness file tat controls the trend kernel to swapping (default is 60)
	(This is my choice for systems where I have swap partitions, desktop, laptops...)

	```console
	# echo 10 > /proc/sys/vm/swappiness
	```

	We can disable the swapping with **swapoff* 
	(This is my choice for systems where I don't have swap partitions as virtual machines)
	```
	# swapoff -a
 	```

- Disable unused services
	If we are using systemd we can disable services on boot by running:

	```console
	# systemctl disable service-name
	```

	We also can use the text-based console tool **sysv-rc-conf** to disable (or enable) services for each runlevel .



_Enjoy! ;)_
