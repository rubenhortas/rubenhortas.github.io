---
title: Keep alive ssh sessions
date: 2022-06-17 00:00:01 +0000
categories: [ssh]
tags: [ssh, howto]
---

Sometimes firewalls time out idle sessions after a certain period of time.  
We can avoid have our SSH sessions killed with a few options to keep alive ssh sesions.  
We can configure the keep alive on the server side or on the client side.  

* On the client side:  
We need to edit the ~/.ssh/config file and add or uncomment the following lines:

```shell
ServerAliveInterval 300
ServerAliveCountMax 2
```

* On the server side:  
We need to edit the /etc/ssh/sshd_config file and add or uncomment the following lines:

```shell
ClientAliveInterval 300
ClientAliveCountMax 2
```

Finally we need to restart our sshd server.

Thanks to [Rodrigo Rega](https://rodrigorega.es/)!

_Enjoy! ;)_
