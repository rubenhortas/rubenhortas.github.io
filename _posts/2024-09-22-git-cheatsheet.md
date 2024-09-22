---
title: git cheatsheet
date: 2024-09-22 00:00:01 +0000
categories: [git, cheatsheets]
tags: [git, cheatsheets]
---

Nowadays, git must be the most widely used source control. 
Although [almost] all IDEs implement it. 
Since git is very extensive, each IDE integrates its own set of actions.
This means tha some actions will behave [slightly] differently between IDEs. 
Some actions will be in some IDEs and not in others, and some actions will not be implemented in any IDE.
If our IDE does not implment the action/actions we need we will have to resort to the console (or CLI). 
It is not uncommon to have to resort to the console, especially when things go wrong.
Since I use git a lot from the CLI, my colleagues often come to me to ask me commands or how to fix some error.
Over time I have realized that some of these questions are recurring, so I have decided to leave this resume of git commands in form of cheatsheet here.

For me, the best source of git documentation is the official one. 
In the [git book](https://git-scm.com/book/en/v2) you will find **ALL** about git. 
There you can find more extensive information about the following commands, and other commands.

>In this cheatseet I will following the next conventions:
>
> *[parameter]*: It means that it is an optional parameter.
>
> *(parameter1,parameter2)*: It means a group of parameters.
>
> *[parameter1|parameter2]*: It means that the are optional and exclusive parameters.
>
> *(parameter1|parameter2)*: It means that the are mandatory and exclusive parameters.
>
> $variable: It means that it a variable and should be replaced by your specific value.
>
{: .prompt-info}

## Set user credentials

* `git config --global user.name $name`
* `git config --global user.email $mail`

## Create a repository

* `git init` Creates a new Git repository in the current directory.
* `git clone $repository` Downloads an existing Git repository from a remote URL.

## Changes

* `git add (.|file)` Adds files to the staging area for the next commit.
* `git commit -ma $message` Creates a new commit with a message.
* `git fetch` Retrieves updated branch information from the remote repository without merging them locally.
* `git pull`Fetches updates from the remote repository and attempts to merge them into the current local branch.
* `git push` Uploads local commits to the remote repository.
* `git reset [--hard] [file|commit]` Moves the HEAD pointer to a specific commit (hard removes local changes since then).
* `git tag [-a|-d] $tag` Creates/deletes a tag for a specific commit.
* `git blame` Shows who last modified each line of a file.
* `git cherry pick`Selectively applies a commit from another branch to the current branch.
* `git revert` Creates a new commit that undoes the changes introduced by another commit.

>Be careful with `git revert`
{: .prompt-info}


## Stashes

* `git stash [list|pop|drop]` Temporarily saves/lists/applies/discards uncommitted changes.

## Repository status

* `git log [--follow $file]`  Shows a history of commits.
* `git diff` Shows the difference between the working directory and the staging area or between commits.
* `git checkout` witches to a different branch or file version.
* `git status` Shows the current state of the working directory.
* `git show $commit` Displays details about a specific commit.
* `git reflog` Shows a history of HEAD.

## Branches

* `git branch`Lists existing branches.
* `git branch $branch-name` Creates a new branch.
* `git checkout $branch-name` or `git switch $branch-name` Switches to an existing branch.
* `git merge` Combines the history of another branch into the current branch.
* `git rebase` Reapplies commits from a branch on top of another branch (potentially rewriting history).
* `git branch -d $branch-name` Deletes a branch (only if it's fully merged).

>Be careful with `git rebase`
{: .prompt-info}

## Aliases

If you are like me, and you use git a lot from the CLI, you will appreciate the following trick.
You can create aliases to group several parameters and speed up ypur work:

* `git config --global alias.tree "--graph --decorate --all --oneline"`

Now, you will have available the following command:

`git tree`

# Squashing

Squashing is a technique that consists of taking n number of commits and merge them into a single commit.
It's used to keep the history clean. 

1. Start an interactive rebase:

  `git rebase -i HEAD~$number_of_commits`

2. Mark commits for squashing: 

  Change the pick keyword to squash or s before the commits you want to combine.

3. Save and exit the editor.

# Update repositories from a bash script

When I make my backups, I want to save an image of all my repositories with the latest updates of all its branches.
I run the following command from a bash script (it will work from any directory) to keep my local repositories up-to-date with changes made in remote repositories:

`git -C $repository fetch --all && git branch --all --track`

*Enjoy! ;)*
