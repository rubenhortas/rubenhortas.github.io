---
title: Firefox DNS over HTTPS
date: 2022-12-09 00:00:01 +0000
categories: [privacy, firefox]
tags: [privacy, firefox, doh, dns, https]
---

When you type a web address or domain name into your address bar (example: [www.mozilla.org](www.mozilla.org)), your browser sends a request over the Internet to look up the IP address for that website.
Traditionally, this request is sent to servers over a plain text connection.
This connection is not encrypted, making it easy for third-parties to see what website you’re about to access.

[DNS-over-HTTPS (DoH)](https://en.wikipedia.org/wiki/DNS_over_HTTPS) works differently.
It sends the domain name you typed to a DoH-compatible DNS server using an encrypted HTTPS connection instead of a plain text one.
This prevents third-parties from seeing what websites you are trying to access. 

[DoH](https://en.wikipedia.org/wiki/DNS_over_HTTPS) increases our privacy and security by preventing eavesdropping and manipulation of DNS data by man-in-the-middle (MITM) attacks.
[DoH](https://en.wikipedia.org/wiki/DNS_over_HTTPS) offers an opportunity to protect our privacy and communications in an untrusted environment.

While [DNS-over-HTTPS (DoH)](https://en.wikipedia.org/wiki/DNS_over_HTTPS), potentially, will give us the ability to conceal DNS requests from ISPs allowing us to circunvent parental controls and legally mandated blockings, we should keep in mind that our ISP is analyzing **all** our traffic. 
And the same could be happen (and usually happens) inside a corporate network.
Also, in most corporate networks we won't be able to activate the DoH options in our browser because our browser will be managed by the corporation.

We also have to keep in mind that our browsing information could be leaked in many ways. For example, with http requests all the communication will be unencrypted, and, in https request the server Name Indication (SNI) is left unencrypted. And the IP is left unencrypted regardless of the implementation.

As always, I leave the decission to activate [DoH](https://en.wikipedia.org/wiki/DNS_over_HTTPS) up to you.

If you want to enable [DNS-over-HTTPS (DoH)](https://en.wikipedia.org/wiki/DNS_over_HTTPS) in firefox, there is a step by step howto on [mozilla's support page](https://support.mozilla.org): [Firefox DNS-over-HTTPS](https://support.mozilla.org/en-US/kb/firefox-dns-over-https)


_Enjoy! ;)_
