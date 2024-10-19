---
title: Python web server
date: 2022-06-03 00:00:01 +0000
categories: [python, web server]
tags: [python, web server, web, server, programming, howto]
---

If we want to test a web site or share some files with another person we can have a web server running quickly with python only running one command:

* Run a web server on the current directory:

```python
$ python3 -m http.server
```

* Run a web server on other directory:

```python
$ python3 -m http.server --directory /other/directory
```

* Run a web server on another port

Python web server listens to port 8000 by default.
The default port can be overriden by passing the desired port number as argument:

```python
$ python3 -m http.server 443
```

_Enjoy! ;)_
