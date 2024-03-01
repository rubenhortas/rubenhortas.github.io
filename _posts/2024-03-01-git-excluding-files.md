---
title: Excluding local files in git without creating a .gitignore
date: 2024-03-01 00:00:01 +0000
categories: [programming, git]
tags: [programming, git]
---

Let's suppose that you have cloned a git third-party repository and, locally, inside the repository, you have new (and untracked) files.
These files could be files created by you, by text editors, etc.
Does not matter.
These are the kind of files that you don't want to share with others.
Or, simply, you don't have permissions to share these files and it's quite annoying for you stash them (or delete them) every time you pull the last version.

For example, you have cloned the [github seclists repository](https://github.com/danielmiessler/SecLists) and you want to have the `rockyou.txt` file umcompressed in its path.
You will not share it with others, you don't have permissions to upload it to the repository. 
Even if you had permissions to upload it to the repository, it will not make sense, since its compressed version it's already uploaded.
So, you want to ignore your local `rockyou.txt` file.

Besides, you want exclude these files without create a `.gitignore` file.
Why?
Because it's not a change you will share with others, because you don't want to see these changes on your shell prompt or because any other reason.

You can create rules that are not committted with the repository adding them to the `.git/info/exclude` file inside the repository in the same way you would add it to `.gitignore`.
For example, if you had the [github seclists repository](https://github.com/danielmiessler/SecLists) cloned into `/opt`, you had to edit your `/opt/SecLists/.git/info/exclude` file (create it if it doesn't exists) and add:

```
rockyou.txt
```

That easy. 
Now your new local (and untracked) files will no longer be a nuisance.

*Enjoy! ;)*