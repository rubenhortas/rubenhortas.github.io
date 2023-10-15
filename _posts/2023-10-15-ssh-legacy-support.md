---
title: SSH legacy support
date: 2023-10-15 00:00:01 +0000
categories: [gnu/linux, administration]
tags: [gnu/linux, administration, ssh]
---

Sometimes we found ourselves in the need of connect from, or to, old devices.
With the passage of time it may that these devices and/or their applications became non upgradeable.
Meanwhile, SSH keeps evolving and disabling insecure key exchange methods (among others).
Of course, keep using these devices and applications, is not the most recommended from a security point of view, but, sometimes there's no alternative.

In my case I have an old phone with and old application which uses SSH.
Both works very well, and neither can be updated.
And I don't think in replace either ot them in the short term.
So, after a fresh install, or after replacing the SSH config file in an update, I know I have to add legacy support to SSH when I see the following message:

> Unable to negotiate with xxx.xxx.xxx.xxx port 22: no matching key exchange method found. Their offer: diffie-hellman-group-exchange-sha1,diffie-hellman-group14-sha1

The solution is allow logins from systems that do not support secure key exchange methods by explicitly enabling insecure key exchange methods.
I have to highlight "**insecure**" here.
Anyway, to enable it, we need to add the following lines to the `/etc/ssh/sshd_config` file:

```shell
KexAlgorithms +diffie-hellman-group1-sha1,diffie-hellman-group14-sha1,diffie-hellman-group-exchange-sha1
HostKeyAlgorithms +ssh-rsa,ssh-dss
PubKeyAcceptedAlgorithms +ssh-rsa,ssh-dss
```

*Enjoy! ;)*
