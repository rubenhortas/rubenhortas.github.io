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

Since I use git a lot from the CLI, my coworkers often come to me to ask me commands or how to fix some error.
Over time I have realized that some of these questions are recurring, so I have decided to leave this resume of git commands in form of cheatsheet here.  

For me, the best source of git documentation is the official one.  
In the [git book](https://git-scm.com/book/en/v2) you will find **ALL** about git.  
There you can find more extensive information about the following commands, and other commands.  
You also can find all the information in the git manual: `$ man git`.

>In this cheatseet I will following the next conventions:
>
> * *[parameter]*: It means that it is an optional parameter.
>
> * *(parameter1,parameter2)*: It means a group of parameters.
>
> * *[parameter1|parameter2]*: It means that the are optional and exclusive parameters.
>
> * *(parameter1|parameter2)*: It means that the are mandatory and exclusive parameters.
>
> * $variable: It means that it a variable and should be replaced by your specific value.
>
{: .prompt-info}

## Configure user settings

* `git config --global user.name $name`
* `git config --global user.email $mail`

## Create a repository

* `git init` Creates a new Git repository in the current directory.
* `git clone $repository` Downloads an existing Git repository from a remote URL.

## Changes

* `git add (.|file)` Adds files to the staging area for the next commit.
* `git rm [-rf] --cached ($file|$directory)` Deletes a file or director from the staged area. 
* `git commit -ma $message` Creates a new commit with a message.
* `git fetch` Retrieves updated branch information from the remote repository without merging them locally.
* `git pull`Fetches updates from the remote repository and attempts to merge them into the current local branch.
* `git checkout $branch -- $file` Gets a file from a branch.
* `git push [--follow-tags]` Uploads local commits to the remote repository.
* `git blame` Shows who last modified each line of a file.
* `git cherry pick`Selectively applies a commit from another branch to the current branch.

### Tags

* `git tag [-a|-d] $tag` Creates/deletes a local tag for a specific commit.
* `git tag -l [$regex]` Lists tags.
* `git show-ref --tags` Shows commits for all tags.
* `git rev-list -n1 $tag` Shows which commit has a certain tag.
* `git push --delete origin $tag %% git tag -d $tag` Deletes a remote tag and, then, deletes the local tag.

### Undo changes

* `git checkout [file]` Discard uncommitted changes
* `git reset [--hard] [file|commit]` Moves the HEAD pointer to a specific commit (hard removes local changes since then).
* `git reset --soft HEAD~1` Undoes the last commit.
* `git revert` Creates a new commit that undoes the changes introduced by another commit.

>Be careful with `git revert`
{: .prompt-warning}

### Squashing

Squashing is a technique that consists of taking n number of commits and merge them into a single commit.
It's used to keep the history clean. 

1. Start an interactive rebase:

    `git rebase -i HEAD~$number_of_commits`

2. Mark commits for squashing: 

    Change the *pick* keyword to *squash* or *s* before the commits you want to combine.

3. Save and exit the editor.

## Stashes

* `git stash list` Temporarily lists uncommitted changes.
* `git stash push` Temporarily saves uncommitted changes.
* `git stash pop $stash` Temporarily applies uncommitted changes.
* `git stash drop $stash` Temporarily discards uncommitted changes.

## Branches

* `git branch`Lists existing branches.
* `git branch $branch-name` Creates a new branch.
* `git checkout -b $branch-name` Creates a new branch and switches to it.
* `git push --set upstream $remote-repository $branch-name` Create a local branch remotely.
* `git checkout $branch-name` or `git switch $branch-name` Switches to an existing branch.
* `git merge` Combines the history of another branch into the current branch.
* `git branch [-d|-D] $branch-name` Deletes a local branch. `-d` deletes the branch if it has already fully merge, `-D` deletes the branch regardless of its merged status.
* `git push $origin --delete $branch-name` Deletes a remote branch.
* `git rebase` Reapplies commits from a branch on top of another branch (potentially rewriting history).

>Be careful using `git rebase`
{: .prompt-warning}

### Rename a local branch

`git branch -m $new_name`

### Rename a local and a remote branch

```shell
git branch -m $new_name # Rename the local branch
git push origin $new_name # Push the new branch to the remote
git push origin --delete $old_name # Delete the old remote branch
```

### Recover a deleted branch

1. Find the SHA1 for the commit of your deleted branch: `git reflog`

2. `git checkout $SHA1`

3. `git checkout -b $branch-name`

## Repository status

* `git status` Shows the current state of the working directory.
* `git show $commit` Displays details about a specific commit
* `git log [--follow $file]`  Shows a history of commits.
* `git reflog` Shows a history of HEAD.
* `git diff` Shows the difference between the working directory and the staging area or between commits.
* `git diff stash@{0}^! -- [$file]` Compares the changes in the most recent stash entry (stash@{0}) with the current working directory/file.
* `git diff $branch1 $branch2 -- $file` Compares the file between two branches.
* `git diff HEAD^ HEAD` Compares the changes between the current commit (HEAD) and its immediate parent commit (HEAD^)..

## Submodules

* `git submodule update --init --remote` Downloads submodules.

## Aliases

If you are like me, and you use git a lot from the CLI, you will appreciate the following trick.
You can create aliases to group several parameters and speed up ypur work:

* `git config --global alias.tree "log --graph --decorate --all --oneline"`
* `git config --global alias.fulltree "log --graph --full-history --all --color --pretty=format:'%x1b[31m%h%x09%x1b[32m%d%x1b[0m%x20%s'`

Now, you will have available the following commands:

`git tree`
`git fulltree`


## Update repositories from a bash script

When I make my backups, I want to save an image of all my repositories with the latest updates of all its branches.
I run the following command from a bash script (it will work from any directory) to keep my local repositories up-to-date with changes made in remote repositories:

`git -C $repository fetch --all && git branch --all --track`

# Acknowledgments

* [Rodrigo Rega](https://rodrigorega.es) Thanks a lot for the tips and the `fulltree` alias :)

*Enjoy! ;)*
