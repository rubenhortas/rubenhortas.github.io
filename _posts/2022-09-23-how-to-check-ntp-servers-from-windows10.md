---
title: How to check NTP servers from Windows 10
date: 2022-09-23 00:00:01 +0000
categories: [windows, administration]
tags: [windows, ntp, sntp, nis, time, howtos]
---

Sometimes we need to retrieve the date from a NTP [Network Time Protocol](https://es.wikipedia.org/wiki/Network_Time_Protocol) 
server to record it somewhere.

The first thing is to know if the server is working and reacheable from our computer (yep, firewalls still works).
Then we can check the returned date format to adapt to our needs.

The most common thing is that we have to use a [NIST](https://www.nist.gov/pml/time-and-frequency-division/time-distribution/internet-time-service-its) server or a [SNPT](https://es.wikipedia.org/wiki/Network_Time_Protocol#SNTP) server. And, at the moment of checking them and programming, there is subtle differences between them.

# Checking a NIST server

Check a NIST server is very straightforward. These servers listens on port 13, and responds to TCP or UDP requests.
We can do a simple telnet or a netcat to check it. For example, if we wanted to check if time-c.nist.gov will work for us:

windows > run > cmd

```shell
telnet time-c.nist.gov 13
```

```shell
netcat time-c.nist.gov 13
```

# Checking a SNTP server

Check a SNTP is not as straightforward as check a NTP server. SNTP servers uses the UDP protocol and the 123 port.
Telnet or netcat won't work for us... 
But we can use [w32tm](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/ff799054(v=ws.11)). 
For example, if we wanted to check if ntp02.oal.ul.pt will work for us:

windows > run > cmd

```shell
w32tm /stripchart /computer:ntp02.oal.ul.pt /dataonly /samples:1
```

_Enjoy! ;)_
