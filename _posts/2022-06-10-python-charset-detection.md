---
title: Python charset detection
date: 2022-06-10 00:00:01 +0000
categories: [python, charset detection]
tags: [python, charset, programming, howto]
---

We can use the [chardet](https://github.com/chardet/chardet) module to detect the charset (or character encoding) of a file.  
This can come handy when we need to analyze a text file.  
To install [chardet](https://github.com/chardet/chardet):  

```shell
$ sudo pip3 install chardet
```

Now we have a command line tool called chardetect to find files charset:

```python
chardetect file.txt
file.txt: ascii with confidence 1.0
```

_Enjoy! ;)_
