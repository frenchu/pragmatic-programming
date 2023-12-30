+++
date = 2023-12-22
publishDate = 2023-12-27
title = "Multiple SSH keys setup for git"
description = "Using two GitHub accounts on the same workstation efectively"
tags = [
  "git", "github", "ssh"
]
categories = [
  "tools"
]
images = ["/images/git.jpg"]
+++

I always configure git to use SSH for connection. And I don't share private keys among instances. I belive this is the best practice.

As a contractor, I need to perform my work duties on my personal equipment from time to time.
Organisations quite often hosts its repositories on [github.com](https://github.com/).

GitHub in its [official docs](https://docs.github.com/en/account-and-profile/setting-up-and-managing-your-personal-account-on-github/managing-your-personal-account/merging-multiple-personal-accounts) encourage users to use the same personal account for both private use and when working for an organisation.
However, quite often I want or even have to seperate work account and private one from each other.

The thing is I can't upload the same key for two different user accounts in GitHub.
It makes sense because GitHub identifies a user by SSH key, so the keys has to be unique.
It is also true for other similar services like BitBucket.

I have to generate two keys on my personal workstation in this case - one for work account and another for private account.
It causing some difficulties while working with git. I need to tell git somehow which key I want to use at the moment. 

Below I described how to configure git to juggle SSH keys with an ease.

{{< figure alt="Git - files of office paper" src="/images/git.jpg" caption="Git repository" >}}

## Inline git command configuration

The simplest way is to inline a configuration parameter when executing git command.
The parameter can change default command for SSH to pass custom key file path. 


You can use environment variable

```shell
GIT_SSH_COMMAND="ssh -i /path/to/key" git pull
```

or command line parameter

```shell
git -c core.sshCommand="ssh -i /path/to/key" git pull
```

If you want to switch from 'home' mode to 'work' mode for the whole terminal session, 
you can export the varibale

```shell
export GIT_SSH_COMMAND="ssh -i /path/to/key"
```

and the other key will be used with subsequent github commands.

Or you can set up a bash alias, something like

```shell
alias gitw 'git -c core.sshCommand="ssh -i /path/to/key"'
```

and then do for example

```shell
gitw pull
```

## IncludeIf directive with gitdir

This setup is more complex, but on the other hand you don't need to set any variable each time you need to switch or remember to use alias.

If you have a clean separation between work and private projects on the directory structure level, you can go with this setup.

First edit your `~/.gitconfig` file to add `includeIf` directive:

```
[user]
    name = "Pawel Weselak"
    email = priv@pawelweselak.com
[includeIf "gitdir:~/devel/work/"]
    path = ~/devel/work/.gitconfig
```

This will tell git to use additional configuration for repos stored in `~/devel/work/` directory.

It will take additional config from `~/devel/work/.gitconfig`.
The path of the included config file can be freely changed.
If you use relative path, it will be relative to the location of `.gitconfig`.

In the included file change SSH command and let's say email:

```
[core]
    sshCommand = "ssh -i /path/to/key"
[user]
    email = pawel.weselak@example.com
```

To verify how new setup works execute command:

```shell
git config --show-origin --get user.email
```

It should provide different outputs depending on the location of the repo inside or outside `~/devel/work/` directory.

The problem with this configuration is that it won't work when you want to clone a new repository.
It's because directories under `~/devel/work/` or the directory itself have to be an existing git repositories.

You can do `git init` under `~/devel/work/` to create a dummy repo, but it would be rather artificial and hacky.

Execute test commands below, to have a better understanding of the problem. 

```
> cd
> git config --show-origin --get user.email
file:/home/pweselak/.gitconfig priv@pawelweselak.com

> cd ~/devel/work/
> git config --show-origin --get user.email
file:/home/pweselak/.gitconfig priv@pawelweselak.com

> mkdir test_repo && cd test_repo
> git config --show-origin --get user.email
file:/home/pweselak/.gitconfig priv@pawelweselak.com

> git init
> git config --show-origin --get user.email
file:/home/pweselak/devel/work/.gitconfig pawel.weselak@example.com
```

You can see that only after repo initialisation a new config has been applied.
Executing `git config` with `--show-origin` provides an information where the config comes from.

## IncludeIf directive with hasconfig

After digging in git documentation and doing some tests, I've found the way how to overcome difficulty described above.

We can use `hasconfig:remote` in `includeIf` directive. Take a look on my `~/.gitconfig`:

```
[user]
    name = "Pawel Weselak"
    email = priv@pawelweselak.com
[includeIf "hasconfig:remote.*.url:git@github.com:WorkOrg/**"]
    path = ~/devel/work/.gitconfig
```

Contents of `~/devel/work/.gitconfig` remains the same.

Now all repos cloned from `WorkOrg` account/organisation on GitHub will use different configuration.
The best thing is when git clones the repo, it initialises the repo config with remote url and 
only then the custom configuration is applied. So when the repo is fetched the right ssh key is used.
Thanks to that the `git clone` works smoothly.

In addition to that, you don't need to keep your directory structure neat and tidy.
The glob pattern in the remote url can be customized and more complex.

You can execute test commands, to check how git behaves now.

```
> git clone git@github.com:frenchu/hello-world.git
> cd sample-repo
> git config --show-origin --get user.email
file:/home/pweselak/.gitconfig priv@pawelweselak.com

> cd ..
> git clone git@github.com:WorkOrg/sample-repo.git
> cd sample-repo
> git config --show-origin --get user.email
file:/home/pweselak/devel/work/.gitconfig pawel.weselak@example.com
```

One caveat of `hasconfig` is that you need to have quite recent version of git installed - 2.36.0 or above.

## Final conclusions

Presented three methods of configuring git may be useful also in other scenarios.
You can not only apply different SSH keys, but also for instance sign commits with separate PGP signatures.

Leave your ideas in the comments section below.

Happy hacking!

## Web resources

* [Conditional includes, Git documentation](https://git-scm.com/docs/git-config#_conditional_includes)
* [Multiple SSH Keys settings for different github account Gist](https://gist.github.com/jexchan/2351996)
