---
title: Customizing default directory names on GNU/Linux
date: 2023-09-02 00:00:01 +0000
categories: [gnu/linux, administration]
tags: [gnu/linux, administration, directories, xdg]
---

Sometimes we don't like default named directories as "Desktop" or "Downloads" and we prefer lowercase namings as "desktop" or "downloads", name them in another language, or, simply, give them another name.
Some applications create default named directories and it's a pain to have duplicated directories and have to delete them later.
Lucky for us, we can change this behavior gobally editing the `/etc/xdg/user-dirs.defaults` file or locally editing the `~/.config/user-dirs.dirs`.

As I prefer make the changes globally, I will edit the `/etx/xdg/user-dirs.defaults`.

From [manpages.debian.org](https://manpages.debian.org/bookworm/xdg-user-dirs/user-dirs.defaults.5.en.html):

> The /etc/xdg/user-dirs.defaults file is a text file that contains the default values for the XDG user dirs which are used by the xdg-user-dirs-update command.
>
> This file contains lines of the form
>
> NAME=VALUE
>
> The following names are recognised:
> DESKTOP
> DOWNLOAD
> TEMPLATES
> PUBLICSHARE
> DOCUMENTS
> MUSIC
> PICTURES
> VIDEOS
>
> The values are relative pathnames from the home directory and will be translated on a per-path-element basis into the users locale.
>
> Lines beginning with a # character are ignored.

So, we can edit `/etc/xdg/user-dirs.defaults` and customize our directories, e.g:

```
# Default settings for user directories
#
# The values are relative pathnames from the home directory and
# will be translated on a per-path-element basis into the users locale
DESKTOP=desktop
DOWNLOAD=downloads
TEMPLATES=templates
PUBLICSHARE=public
DOCUMENTS=documents
MUSIC=music
PICTURES=pictures
VIDEOS=videos
# Another alternative is:
#MUSIC=Documents/Music
#PICTURES=Documents/Pictures
#VIDEOS=Documents/Videos
```

If the xdg-user-dirs-update.service is enabled, will keep the directories updated at the beginning of each login session.

*Enjoy! ;)*
