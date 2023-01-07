--e
t Mozilla wiki page summarising the current anti-phishing implementations and possible plans for the futureitle: Avoid punycode attacks on firefox
date: 2023-01-13 00:00:01 +0000
categories: [hardening, firefox]
tags: [hardening, firefox, punycode]
img_path: /assets/img/posts/
---

 # What is punycode?

[Punycode][1] is a representation of Unicode with the limited ASCII character subset used for Internet hostnames.
Using [punycode][1], host names containing Unicode characters are transcoded to a subset of ASCII consisting of letters, digits and hypens, which is called LHD (Letter Digit Hypen) subset.
The DNS (Domain Name System) standars recommend the use of the LDH subset of ASCII.
the [punycode][1] syntax is a method of encoding string containign Unicode characters into the LDH subset of ASCII.
 
In other words, [punycode][1] is a method to convert words that can't be written in ASCII (because they use Unicode characters) into ASCII encoding words for use as domain names.
For example: 

| UTF8                   | Punycode                           | 
----------------------------------------------------------------
| rubénhortas.github.io  | https://xn--rubnhortas-d7a.github.io


# What is a punycode attack?

A punycode attack is a type of [homograph attack][2].
Some unicode characters are very similiar to ASCCI characters, making them difficult to distinguish with the naked eye. 

As the security researcher Xudong Zheng demonstrates on his post [Phishing with Unicode Domains](https://www.xudongz.com/blog/2017/idn-phishing/) it's possible to register domains such as "xn–pple-43d.com", which is equivalent to "аpple.com". 
It may not be obvious at first glance, but "аpple.com" uses the Cyrillic "а" (U+0430) rather than the ASCII "a" (U+0061).
 
Visually, the two domains are indistinguishable due to the font used by some browsers.

The Unicode consortium provides a long list of confusable glyphs: [confusableWholeScript.txt](https://unicode.org/reports/tr39/data/confusablesWholeScript.txt)  

# How to avoid punycode attacks on firefox?

This bug was reported to Chrome and Firefox on January 20, 2017.
And was fixed in the Chrome trunk on March 24.

Mozilla has a wiki page explaining how Firefox decides whether to display a given IDN label (a domain name is made up of one or more labels, separated by dots) in its Unicode (i.e. normal) or punycode (i.e. gobbledigook) form: [Mozilla IDN Display Algorithm](https://wiki.mozilla.org/IDN_Display_Algorithm).

The point is that, with my system configuration, in Firefox I'm unable to distinguish the real [https://www.apple.com](https://www.apple.com) from the Xudong Zheng example [https://www.аррӏе.com/](https://www.аррӏе.com/). While chrome and chromium shows [https://www.xn--80ak6aa92e.com/](https://www.xn--80ak6aa92e.com/) instead [https://www.аррӏе.com/](https://www.аррӏе.com/).

> I also checked brave in android and it shows [https://www.xn--80ak6aa92e.com/](https://www.xn--80ak6aa92e.com/) too.
{: .prompt-info}

Although I understand the need and utility of [punycode][1] I don't need this behaviour, and I want it off to reduce my expousure to [homograph attacks][2]. 
To force Firefox to display punycode names we should open the about:config tab and set

```
network.IDN_show_punycode = true
```

[1]:https://en.wikipedia.org/wiki/Punycode
[2]:https://en.wikipedia.org/wiki/IDN_homograph_attack

*Enjoy! ;)*
