---
title: Organize your dotfiles with git
date: 2024-10-13 00:00:01 +0000
categories: [git, dotfiles]
tags: [git, dotfiles]
---

## What are dotfiles?

Dotfiles are files where the programs store their configuration.
Dotfiles are named that way because most of them starts with  a dot ".", although, nowadays, it's common to use this term to refer to any configuration file.

When you install a new program, you spend a lot of time reading documentation and trying and setting options, until fits you well.
Some examples of programs that uses dotfiles are: bash, zsh, git, tmux, vim...
In addition to these, I like to save another dotfiles (like .xprofile) and another configuration files (awesomewm, neovim, mplayer, yt-dlp, conky...).

Saving these dotfiles (or configuration files) will save you a lot of time, and will allow you to mantain a consistency between computers.
And I love these two things.

Let's say that you uninstalled some of these programs, or you are doing a fresh install of your operating system, or, maybe, you are installing some of these programs in another computer.
After install these programs, you will only have to override their dotfiles (or config files) and everything will be to your liking and the same between systems/computers (if applicable).

I have many computers, all but one, using gnu/linux.
I keep some configs of all of them, some scripts or batch files (yep, it's very useful backup these too) and even the wallpaper.

Is true that we can do this with a simple backup in any hard drive, but let's face it, we are programmers; and have the dotfiles backuped in a remote (and always available) git server (as github) is very comfortable.

## The "all in one" approach

This was my first approach.
This approach consists of having one repository with only one branch. Then we will structure everything in folders.

Once we had cloned our repository, we will delete our local dotfiles (or config files) and make symlinks to the files in the repository.

My choice is to do hard links for files and soft links for the folders, but as you like ;)

We will have something like this:

```shell
dotfiles
├── linux
│   ├── common
│   │   ├── configs
│   │   │   ├── aliases
│   │   │   ├── awesome
│   │   │   ├── nvim
│   │   │   ├── .tmux
│   │   │   ├── .vimrc
│   │   │   └── yt-dlp.conf
│   │   ├── .mplayer
│   │   └── scripts
│   │       └── script1
│   ├── desktop
│   │   ├── configs
│   │   │   ├── .bashrc
│   │   │   ├── conky.conf
│   │   │   ├── .gitconfig
│   │   │   ├── .xprofile
│   │   │   └── .zshrc
│   │   ├── scripts
│   │   │   ├── script1
│   │   │   └── script2
│   │   └── wallpaper
│   │       └── wallpaper.jpg
│   ├── laptop1
│   │   ├── configs
│   │   │   ├── .bashrc
│   │   │   ├── .gitconfig
│   │   │   ├── .xprofile
│   │   │   └── .zshrc
│   │   ├── scripts
│   │   │   ├── script1
│   │   └── wallpaper
│   │       └── wallpaper.jpg
│   └── laptop2
│       ├── configs
│       │   ├── .bashrc
│       │   ├── .gitconfig
│       │   ├── .xprofile
│       │   └── .zshrc
│       ├── scripts
│       │   ├── script1
│       │   └── script2
│       │   └── script3
│       └── wallpaper
│           └── wallpaper.jpg
└── windows
    ├── configs
    │   ├── config1.cfg
    │   └── config2.cfg
    ├── scripts
    │   ├── batch1.bat
    │   └── batch2.bat
    └── wallpaper
        └── wallpaper.jpg
```

### Pros

* Very easy to mantain.

  One repository, one branch. 
It's very easy to keep track of the changes.
You have the same repository and the same branch in all of your machines.

* Common files are easily synchronized

  If we use a `common` directory to store the dotfiles that are the same in all the machines, we can keep our common configuration files easily synchronized.

### Cons

* Fragmentation

  Over time we will have fragmented configurations (if we use the common folder).
In the worst case, we will get rid of the `common` directory.

* Useless configurations

  Wether we have it or not the `common` folder, we will have a lot of files (configurations, scripts, wallpapers...) from other computers that will be, mostly, useless for us in the current computer.

## The "m-branches" approach

This is my current approach, it's similar to the "all in one" approach.
We will have one repository, but one branch per machine (that's why the "m" in the name).

Each branch will contain only the dotfiles for one computer.
We will have something like this:

```shell
dotfiles/desktop
├── configs
│   ├── aliases
│   ├── awesome
│   ├── .bashrc
│   ├── conky.conf
│   ├── .gitconfig
│   ├── .mplayer
│   ├── nvim
│   ├── .tmux
│   ├── .vimrc
│   ├── .xprofile
│   ├── yt-dlp.conf
│   └── .zshrc
├── scripts
│   ├── script1
│   └── script2
└── wallpaper
    └── wallpaper.jpg
```

### Pros

* We will have only the files related to the current machine.

### Cons

* Management can get a little complicated

  If we make a change to a file that we want to push to other machines, we will have to do it manually on the other branches.
  
  If we create a file on one machine that we want to have on other machines, we will have to do it manually or propagate it with git to the other branches.

* Branch switching

  If you change branches, while your aren't in the current computer branch, your links will break.
So, If you need to switch branch, for whatever reason, it's better to clone the repository (again) in another location.

## The "StreakyCobra" approach

This is the better approach. Why? Look at this name, how can it not be the best with this name?

I saw this method in a [Hacker news](https://news.ycombinator.com/news) thread: [Ask HN: What do you use to manage dotfiles?](https://news.ycombinator.com/item?id=11070797).
This was [the response given by the user StreakyCobra](https://news.ycombinator.com/item?id=11071754). 

This is going to be by next system (as soon as I feel like reorganizing some machine).
This method is similar to the "m-branches", one repository and one branch per machine, with the difference that **this method does not need extra tools or symlinks**.
So, we will end having something similar to the "m-branches", but more comfortable to manage.

### Creating the repository

If you don't have the repository yet, you need to create it:

```shell
git init --bare $HOME/.git_dotfiles # Creates the new repository. Replace with your own path.
```

### Cloning the repository

If you have the repository, you have to clone it.

If you have your home directory empty (is not the usual thing), you can clone the repository:

```shell
 git clone --separate-git-dir=$HOME/.git_dotfiles https://path/to/repo $HOME
```

> * --separate-git-dir=$HOME/.git_dotfiles: This option specifies that the Git repository configuration should be stored in the directory $HOME/.git_dotfiles instead of the usual .git subdirectory within the cloned repository.
>
> * /path/to/repo: The URL or path to the Git repository that you want to clone.
>
> * $HOME: This is the path where you want to clone the repository.
{: .prompt-info}

If you don't have your $HOME directory empty, you will need to clone it in a temporary directory and then delete the temporary directory:

```shell
git clone --separate-git-dir=$HOME/.git_dotfiles https://path/to/repo $HOME/myconf-tmp
rm -r $HOME/myconf-tmp/
```

### Creating the "dotfiles" alias

We define an alias named "dotfiles" that executes git commands using the specified repository and hides the untracked files.

```shell
alias dotfiles='/usr/bin/git --git-dir=$HOME/.git_dotfiles/ --work-tree=$HOME --showUntrackedFiles no'
```

To make the alias permanent, we need to add it to our shell's configuration file (`.bashrc`, `.zshrc`, etc).

> The creation of the alias `dotfiles` avoids interference when we use the `git` command in other repositories.
>
> StreakyCobra uses an alias called `config`. I prefer `dotfiles`, it's more clear for me.
{: .prompt-info}

### Managing files

Any file can be versioned with normal commands, but you have to replace the `git` command by the alias `dotfiles`.

```shell
dotfiles status
dotfiles add .vimrc
dotfiles commit -m "Add vimrc"
dotfiles push
```

### Cloning the repository in a new machine

The normal cloning will fail if the home directory is not empty.
When we will clone the repository in a new machine, we will have to clone the repository into a temporary directory and then delete this temporary directory.

```shell
git clone --separate-git-dir=$HOME/.git_dotfiles https://path/to/repo $HOME/myconf-tmp
cp $HOME/myconf-tmp/.gitmodules $HOME  # If you use git submodules
rm -r $HOME/myconf-tmp/
alias dotfiles='/usr/bin/git --git-dir=$HOME/.git_dotfiles/ --work-tree=$HOME'
```

### Pros

* No need to create symlinks.
* We will have only the files related to the current machine.

### Cons

* Needs an alias to do not interfere with the `git` command in other repositories.
* We will have to delete a (temporary) directory. Is it really a con?

*Enjoy! ;)*
