+++
publishDate = 2024-02-21
title = "Make your commits more reliable and secure"
description = "Setting up commit signatures"
tags = ["git", "github", "ssh", "digital signatures"]
categories = ["tools", "security"]
series = ["git"]
images = ["/images/signature.jpg"]
+++

Most probably you are aware of SSH private key authentication mechanism. Recent developments in OpenSSH tooling 
enable using SSH private keys also for signing.
There is a debate if it is right decision to add a new capibility to OpenSSH which was designed for other purpose.
Let's leave aside this dispute and just check how we can benefit from SSH signing functionality. 

On a daily basis I use SSH for interactions with git repositories. Both GitHub and GitLab allows to configure SSH keys for
authentication. Moreover, both of them allow set your SSH keys for signing commits. Having your commit signed,
every third party may use your SSH public key to confirm that you are actually the commit author.
You can easily prove your commit atribution. And other way around, imposters will have a hard nut to crack if they
decide to spoof your work.

In this tutorial I want to show how to prepare your environment for commit signing and don't get a headache somewhere between.

{{< figure alt="Signature - desk with documents and inkwell" src="/images/signature.jpg" caption="Commit signature" >}}

## Upload signing key to GitHub/GitLab

Go to your user profile on GitHub or GitLab, whichever you prefer, find SSH/PGP keys page and upload your SSH public key under 'signing' section.
I believe in most of the git hosting services it looks quite similar. If you are using multiple keys, you can add a name to distinguish them.

Remember to keep your private key safe at all times! Don't copy or move your private key between machines.
Better off generate another key for a new device. Remove key from GitHub as soon as it is no longer needed or you lost control over it.

Personally, I use the same key for signing and authentication. Perhaps it should be separated.
Please let me know what you think in the comments section below. ;)

## Configuring git in your local

When a remote server is aware of your public keys, it's time to configure git locally.

Let's start with setting signing key format to SSH

```shell
git config --global gpg.format ssh
```

Now tell git about your signing key (it can be public if you are using ssh-agent)

```shell
git config --global user.signingKey ~/.ssh/id_ed25519.pub
```

Instead of path to a file, you can provide the public key directly in the command line. It needs to be prefixed with `key::`.

Alternatively, you may want to return your key contents in more dynamic way (e.g. via shell script)

```shell
git config --global gpg.ssh.defaultKeyCommand ~/bin/key_provider.sh
```

If you have SSH agent set up, `ssh-add -L` can be very convinient here

```shell
git config --global gpg.ssh.defaultKeyCommand "ssh-add -L"
```

Now you can go to one of your repos and commit anything with your signature

```shell
git commit -S -m "New post"
```

You can sign all new commits and tags by default and don't have to put '-S' option each time

```shell
git config --global commit.gpgsign true
git config --global tag.gpgsign true
```

## Verifying signed commits

To check signature of any given commit do

```shell
git verify-commit <commit_sha>
```

For example

```shell
git verify-commit HEAD
```

Or look into the history which commits are signed

```shell
git log --show-signature
```

You probably will see error messages saying that git is unable to prove commits authenticity. It's because we need to inform git
what keys are allowed to sign commits. In `allowed_signers` file put public keys of the authors who embrace commit signatures,
including yourself ofcourse.

This is how to inculde yourself to "allowed signers" gang

```shell
git config --global gpg.ssh.allowedSignersFile ~/.ssh/allowed_signers
echo "$(git config --get user.email) namespaces=\"git\" $(cat ~/.ssh/id_ed25519.pub)" >> ~/.ssh/allowed_signers
```

GitHub gives convinient access to your public keys under the link

[https://github.com/<YOUR_GH_USERNAME>.keys](https://github.com/frenchu.keys)

You can share it with your teammates to obtain keys for verification of your commits.

If you are using github web interface to merge pull requests, github also signs its commits. You can obtain github public key from the url

https://github.com/web-flow.gpg

It is not SSH key, though. GitHub signs commits with PGP key which is part of a different story.
Hopefully, I will cover the subject in another post. :)

## Additional resources

- https://docs.github.com/en/authentication/managing-commit-signature-verification
- https://docs.gitlab.com/ee/user/project/repository/signed_commits/ssh.html
