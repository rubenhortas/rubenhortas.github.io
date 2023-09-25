---
title: DNS hardening with Quad9
date: 2023-09-24 00:00:01 +0000
categories: [hardening, dns]
tags: [hardening, dns, quad9]
---

Nowadays, in times when malware proliferates, who isn't worried about clicking by mistake (or not) on a malicious link?
It's more, who isn't worried about a family member or friend, with less computer knowledge, clicking on a malicious link?
A wrong click and all your network could be compromised. All your machines...
Even if only one machine was compromised, I have to admit that it's not one of the things I would like to suffer...

There's a few ways to block malicious domains, one of them is [Steven Black's hosts](https://github.com/StevenBlack/hosts).
It's a great solution, but it has some downsides.
First of all, you need to install it in every machine, generate custom host files and take care of keeping it updated (manually or scripting and scheduling a task).
This could be ok if you have few machines and you have full access whenever you want to them.
But, even so, may cause performance loss on some cases (above all on Windows machines).

So, even I think [Steven Black's hosts](https://github.com/StevenBlack/hosts) it's a great solution, and I use it myself on some machines, what else we can do?
We can protect ourselves through our DNS Server.
If our DNS server takes care of malware, phishing, spyware, botnets and other threats, we won't have to!
In the best case, if we had machines that never connect to another networks, we will only need change the DNS server on our router (if the router allows it).
But, I don't recommend **only** this, because, what is normal is that we have laptops and mobile devices that connects to many networks.
Device DNS server configuracion usually take precedence over the router's configuration.
So, we will use our custom server although the router doesn't allow the change and in any network.
So, I recommend to change the DNS server in our router (if the router allows it) **AND** in every machine that we have.
Setting a custom DNS server on the router will protect the devices that takes the router's DNS server by default (such as a new -and unconfigured- computer, or the laptop and/or mobile phone of a family member or friend invited to our house).
The proccess to set a custom DNS server differs quite a bit between Windows, GNU/Linux and Android, and between their versions, so I'm not going to go into how do it.

The advantages of changing the DNS server, are that we only have to change it one time for each machine and for the router (if the router allows it).
And we can keep [Steven Black's hosts](https://github.com/StevenBlack/hosts) as a second layer protection.
And, even if we can keep it as updated as we want, we can sleep a bit more peacefully knowing that we are a little safer.

# Quad9

The DNS server I chose to protect myself is [Quad9](https://www.quad9.net/).
According to [Wikipedia](https://en.wikipedia.org/wiki/Quad9):

>Quad9 is a global public recursive DNS resolver that aims to protect users from malware and phishing. 
>Quad9 is operated by the Quad9 Foundation, a Swiss public-benefit, not-for-profit foundation with the purpose of improving the privacy and cybersecurity of Internet users, headquartered in Zurich.
>It is the only global public resolver which is operated not-for-profit, in the public benefit.
>...
>Several independent evaluations have found Quad9 to be the most effective (97%) at blocking malware and phishing domains.
>...
>The domains which are filtered are not determined by Quad9, but instead supplied to Quad9 by a variety of independent threat-intelligence analysts, using different methodologies.
>Quad9 was the first to use standards-based strong cryptography to protect the privacy of its users' DNS queries, and the first to use DNSSEC cryptographic validation to protect users from domain name hijacking
>Quad9 protects users' privacy by not retaining or processing the IP address of its users, and is consequently GDPR-compliant.

And, according to [Quad9](https://www.quad9.net/):

>The system uses threat intelligence from more than a dozen of the industry’s leading cybersecurity companies to give a real-time perspective on what websites are safe and what sites are known to include malware or other threats.

So, it's seems a great solution.
Once we have seen the why and the how, all that remains is to set [Quad9's](https://www.quad9.net/) DNS servers as our new custom DNS servers:

```
# Quad9's DNS servers for IPv4: 9.9.9.9, 149.112.112.112
# Quad9's DNS servers for IPv6: 2620:fe::fe, 2620:fe::9
```

*Enjoy! ;)*