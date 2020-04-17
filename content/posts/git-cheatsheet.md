+++ 
date = 2019-01-24
publishDate = 2020-04-16
title = "Git cheatsheet"
description = "Frequently used git commands"
tags = [
    "git",
    "git flow"
]
categories = [
    "tools"
]
+++

## Intro

During the years of experience with git I have noted aside many commands which could be useful.
In this article I present the most frequently used git commands and also configuration steps I always follow in a new work environment.

## Configuration

* User information
		
        $ git config --global user.name "Weselak, Pawel"
        $ git config --global user.email "nospam@pawelweselak.com"
        
* SSH keys

        $ ssh-keygen -t rsa -b 4096 -C "nospam@pawelweselak.com"
        $ eval "$(ssh-agent -s)"
        $ ssh-add ~/.ssh/id_rsa

* safe pull settings

        $ git config --global pull.rebase true
        $ git config --global pull.ff only

* clear out local repo from deleted branches

        $ git config --global fetch.prune true

* apply resolved conflicts to remaining rebase steps

        $ git config --global rerere.enabled true 

* push only the current branch

        $ git config --global push.default simple

* fix line breaks on Windows

       $ git config --global core.autocrlf true


## Typical git flow

1. Clone remote repository to your local box

        $ git clone git@github.com:frenchu/hugo-coder.git

2. Add another remote

        $ git remote add --track master upstream git@github.com:luizdepra/hugo-coder.git

3. Get changes from the remote

        $ git fetch upstream
        $ git merge upstream/master

    or

        $ git pull upstream


4. Create topic branch

        $ git branch feature/add-readme
        $ git checkout feature/add-readme

    or

        $ git checkout -b feature/add-readme

    info about the branch

        $ git branch
        $ git branch -a
        $ git branch -vv

5. Commit

        $ git status
        $ git add .
        $ git status
        $ git commit -m "README.md file added"

    optionally edit your commit if you made a mistake

        $ git commit --amend -m "README.md and License files added"
        
    if you want to keep existing commit message do     
        
        $ git commit --amend --no-edit
        
    if you need to undo three commits type
     
        $ git reset --soft HEAD~3

    or use rebase to interactively edit commits

        $ git rebase -i HEAD~3

6. Push

        $ git push origin feature/add-readme

    to set upstream for feature pushes do

        $ git push -u origin feature/add-readme

## Usefull commands

* stash/unstash

        $ git stash
        $ git stash pop

* undo last commit

        $ git reset --soft HEAD~1

* reset local to remote branch

        $ git fetch origin
        $ git reset --hard origin/branch

* rename local branch

        $ git branch -m <oldname> <newname>

* push with tags

        $ git push --follow-tags origin master

* push to another branch

        $ git push origin develop:feature/FR2019_spin_off

* remove remote branch

        $ git push origin --delete <branch_name>

* squash two last commits

        $ git reset --soft HEAD^
        $ git commit --amend

* fix last commit with editing commit message

        $ git commit --amend --no-edit

* editing previous commits

        $ git rebase -i HEAD~3                   # Three commit before
        $ git rebase --interactive 'bbc643cd^'   # Or starting from given hash
        $ git commit --all --amend --no-edit
        $ git rebase --continue

* change the author of the commit

        $ git commit --amend --author="New Me <new@gmail.com>"

* edit first commit

        $ git rebase -i --root

* move recent commits to new branch

        $ git branch new_branch
        $ git reset --hard HEAD~3
        $ git checkout new_branch

* move recent commits to existing branch

        $ git checkout feature
        $ git merge master
        $ git checkout master
        $ git reset --hard HEAD~1

* pull without checkout

        $ git fetch origin master:master

* prune / clean-up

        $ git remote prune origin
        $ git fetch -p && git branch -vv | awk '/: gone]/{print $1}' | xargs git branch -D


## Additional info

* [aliases for git pull/up](http://stackoverflow.com/questions/15316601/in-what-cases-could-git-pull-be-harmful)