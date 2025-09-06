---
title: fail2bangeolocation
date: 2022-07-16 00:00:01 +0000
categories: [personal, portfolio]
tags: [personal, portfolio, development, fail2bangeolocation, fail2ban, geolocation]
img_path: /assets/img/posts/
---

I have an old raspberry pi running some services 24/7 and with (very protected) remote access enabled.
For some reason the raspberry receives a big number of bruteforce attacks trying to log in.
To know from which locations I was being attacked the most I did a python application to geolocate the failed attempts registered by fail2ban.
I learned many things developing this application. The most remarkable, for me, it was the process of packaging the application in order to upload it to PyPI.

The application is called [**fail2bangeolocation**](https://github.com/rubenhortas/fail2bangeolocation), and you can install it via pip.

With [**fail2bangeolocation**](https://github.com/rubenhortas/fail2bangeolocation) you can geolocate all the ips registered by fail2ban, all the ips registered for fail2ban only for a certain service or all the ips contained in a fail2ban log file. And you can group the output by country or by country and city.

## Screenshots
- Output grouped by country
![Output grouped by country](fail2bangeolocation_screenshot_grouped_by_country.png)
_fail2bangeolocation output grouped by country_

- Output grouped by country and city
![Output grouped by country and city](fail2bangeolocation_screenshot_grouped_by_country_and_city.png)
_fail2bangeolocation output grouped by country and city_

You can take a look at the application source code in [https://github.com/rubenhortas/fail2bangeolocation](https://github.com/rubenhortas/fail2bangeolocation)

_Enjoy! ;)_
