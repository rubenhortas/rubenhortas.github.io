---
title: Organize your dotfiles with git
date: 2024-10-13 00:00:01 +0000
categories: [git, dotfiles]
tags: [git, dotfiles]
---

## What are dotfiles?

Dotfiles are files where the programs store their configuration.
Dotfiles are named that way because most of them starts with '.', but nowadays it's common to use this term to refer to any configuration file.

When you install a new program, you spent a lot of time reading documentation, trying and setting options, until fits you well.
Some examples of programs that uses dotfiles are: bash, zsh, git, tmux, vim...
In addition to these, I like to save another dotfiles (like .xprofile) and another configuration files (awesomewm, neovim, mplayer, yt-dlp, conky...).

Saving these dotfiles (or configuration files) will save you a lot of time, and will allow you to mantain a consitency between computers.
And I love these two things.

Let's say that you uninstalled some of these programs, or you are doing a fresh install of your Operating System, or, maybe, you are installing some of these programs in another computer.
After install these programs, you will only have to override their dotfiles (or config files) and everything will be to your liking and the same between systems/computers (if applicable).

I have many computers, all but one, using gnu/linux.
Of all of them I keep some configs, some scripts or batch files (yep, it's very useful backup these too) and even the wallpaper.

## The all in one approach

This was my first approach.
This approach consists of having one repository with only one branch and structure everything in folders.

Once we had cloned our repository, we will delete our local dotfiles (or config files) and make links to the files in the repository.
My choice is to do hard links for files and soft links for the folders, but as you like ;)

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
  If we use a `common` directory to store the dotfiles that are the same in all the machines, we can keep our common configurations easily synchronized.

### Cons

* Fragmentation
  Over time we will have fragmented configurations (if we use the common folder).
In the worst case, we will get rid of the `common` directory.

* Useless configurations
  Wether we have it or not the `common` folder, we will have a lot of files (configurations, scripts, wallpapers...) from other computers that will be, mostly, useless for us in this computer.

## The branches approach

This is my current approach, it's similar to the latest.
We will have one repository, but one branch per machine.

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

* We will have only the files related to the current machine

### Cons

* Management can get a little complicated

  If we make a change to a file that we want to push to other machines, we will have to do it manually on the other branches.
If we create a file on one machine that we want to have on other machines, we will have to do it manually or propagate it with git to the other branches.

* Branch switching

  If you change branches, while your aren't in the computer branch, your links will break.
So, If you need to switch branch, for whatever reason, it's better to clone the repository in another location.

## The StreakyCobra approach

This is the better approach? Why? Look a this name, how can it not be the best with that name?

I saw that approach in a [Hacker news](https://news.ycombinator.com/news) thread: [Ask HN: What do you use to manage dotfiles?](https://news.ycombinator.com/item?id=11070797).
This was [the response given by the user StreakyCobra](https://news.ycombinator.com/item?id=11071754). 

This is going to be by next method (as soon as I feel like reorganizing some machine).
This method does not need extra tools or symlinks.

### Creating the new repository

```shell
git init --bare $HOME/.myconf # Creates the new repository. Replace with your own path.
alias config='/usr/bin/git --git-dir=$HOME/.myconf/ --work-tree=$HOME' # Defines an alias named config that executes git commands using the specified repository.
config config status.showUntrackedFiles not # Do not display untracked files in the status output.
```

The creation of the alias `config` avoids interference when we use the `git` command in other repositories.

### Usage

Basically, you have to replace `git` by `config`.

```shell
config status
config add .vimrc
config commit -m "Add vimrc"
config push
```

### Cloning the repository in a new machine

As the normal cloning will fail if the home directory is not empty, we will have to cloning the repository into a temporary and then delete this repository.

```shell
git clone --separate-git-dir=$HOME/.myconf /path/to/repo $HOME/myconf-tmp
cp ~/myconf-tmp/.gitmodules ~  # If you use Git submodules
rm -r ~/myconf-tmp/
alias config='/usr/bin/git --git-dir=$HOME/.myconf/ --work-tree=$HOME'
```

### Pros

* No need to create links
* We will have only the files related to the current machine

### Cons

* When we clone it in a new machine we have to clone it in a temporary repository and then delete it. Is it really a con?

*Enjoy! ;)*
