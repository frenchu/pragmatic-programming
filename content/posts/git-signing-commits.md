+++ 
draft = true
publishDate = 2024-02-26
title = "Secure your commits"
description = "Commit signing"
tags = [
  "git", "github", "ssh"
]
categories = ["tools", "security"]
series = ["git"]
+++

```
git config --global gpg.format ssh
git config --global user.signingKey ~/.ssh/id_ed25519.pub
git config --global commit.gpgsign true
git config --global tag.gpgsign true
git config --global gpg.ssh.allowedSignersFile ~/.ssh/allowed_signers
echo "$(git config --get user.email) namespaces=\"git\" $(cat ~/.ssh/id_ed25519.pub)" >> ~/.ssh/allowed_signers
git log --show-signature
```
