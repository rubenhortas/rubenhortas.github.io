---
title: Don't add hostkey to shh known_hosts 
date: 2022-10-28 00:00:01 +0000
categories: [ssh, privacy]
tags: [ssh, privacy, known_hosts] 
---

The known_hosts is a file stored in the user home, in ~/.shh/known_hosts.
The use of the known_hosts file is optional, but this file ensures the identities of the hosts and avoids [man in the middle attacks](https://en.wikipedia.org/wiki/Man-in-the-middle_attack) when we connect to a host via ssh.

Everytime we connect to a host via ssh, if is a new host, the system prompts us whether we want to add the remote host to known_hosts file.
If we accept to add the remote host to the known_host file the host's public key is stored in the known_host file and the host becomes a known host.

When we connect to a known host via ssh the system checks the public key of the remote host, and, if there was any change on the remote host public key we would be alerted.
Of course, there are some situatons for which the public key change is legit, but it is our responsability to check it with the server's administrator (or sysadmin).

Altough **using the known_hosts file is the right way to ensure the identity of the remote host and secure our connection**, there are some situations where we may prefer not to add the host to the kwnown_hosts file.
We may prefer not to add it for **privacy reasons** or, for example, if we regularly play [cybersecurity capture the flag games (or ctfs)](https://en.wikipedia.org/wiki/Capture_the_flag_(cybersecurity)) (even if using a virtual machine) we will end with a lot of useless entries on our known_hosts file.

We can avoid add the public key of the new host to the known_hosts file for a single session using: -o "UserKnownHostsFile=/dev/null"

```shell
ssh user@newhost -o "UserKnownHostsFile=/dev/null"
```

But, if we want to avoid add the public keys of new hosts permanently we should update our ssh config (~/.ssh/config to do it at user level or /etc/ssh/ssh_config to do it at system level) and add or comment the following lines:

```
# We can specify the hosts from which the public keys will not be saved with a regex. 
# If you are going to use this option use the strictest regex you can.
# If you want to avoid adding any public key of any host you can skip the Host line
Host *.mydomain.com
  StrictHostKeyChecking no
  UserKnownHostsFile /dev/null
  User foo # For the given user (optional)
  LogLevel QUIET # Will keep the warning message from showing up
```

_Enjoy! ;)_
