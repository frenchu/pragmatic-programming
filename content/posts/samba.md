+++
date = 2024-01-21
publishDate = 2024-01-21
title = "Samba server setup on Ubuntu"
description = "How to quickly setup Samba server on Ubuntu"
tags = [
  "samba", "services", "networking", "admin", "ubuntu"
]
categories = [
  "tools"
]
images = ["/images/samba.jpg"]
+++

Carnival is at the height of the season,
so it's a good time to publish the first post in 2024!

Samba is not only popular dance from Brasil
and music instrument, but also a unix service.

Typically in a home LAN, you need some interoperability 
between your linux and windows machines.
For instance file sharing between home server
and personal computers of the family members.
And one of the simplest solutions for that on Linux is...
Yes, you nailed it, Samba!

Samba, according to its manual, is a collection of programs
that implements Sever Message Block (SMB) protocol
also known as Common Internet File System (CIFS).

In this post I will quickly guide you through Samba suite
installation and configuration on Ubuntu.

{{< figure alt="Samba - carnival in Brasil" src="/images/samba.jpg" caption="Samba" >}}

## Install samba

First, let's install Samba suite.

```bash
sudo apt update
sudo apt install samba
```

## Edit config

Before start sharing any files, you may want to explore config options and tweak the service.
Then restart Samaba daemon.

```bash
sudo nano /etc/samba/smb.conf
sudo service smbd restart
```

## Configure samba account for yourself

You need to set a separate samba password associated with your linux user.
It can be (and even should be) different from your linux password.

Next, add your linux user to the Samba user group. 

```bash
sudo smbpasswd -a $USER
sudo usermod -a -G sambashare $USER
```

## Add accounts for others

If you want to share files with others, configure accounts also for them.
They doesn't use linux and doesn't have own linux username.
Although, to properly configure permissions for the shares, first we need to have a linux account for each user.
And then set up samba passwords and group for the users simirarly to the previous step.
You can lock unix accounts with `passwd` as a security measure.

Let's say you exchange documents only with your significant other. Here's the example:

```bash
sudo adduser karen -G sambashare
sudo smbpasswd -a karen
sudo passwd -l karen
```

## Configure shares

And now the most interesting part. Because of Samba group membership, you can configure shares as a regular user
without sudo. Make sure all users have unix access rights to navigate and read shared files and directories.
It can be easily done by changing group ownership of the files to the `sambashare`.

In the example below only current user (you) may access priv share, while docs are shared with karen.
Access is denided for everyone else.

```bash
sudo chgrp -R sambashare shares/
net usershare add priv /shares/Documents/priv "My Stuff" $USER:f,everyone:d guest_ok=n
net usershare add docs /shares/Documents/docs "Shared Documents" $USER:f,karen:f,everyone:d guest_ok=n
```

Here's a quick explenation of samba ACLs (access control list). Username and access type are joined with colon
and the list elements are separated with coma. Access type can be:

* `f` for full access
* `r` for read-only
* `d` for denied

Add guest_ok=n to disable guest access. `everyone` is a special identifier meaning all samba users.
By default all users have read only access.

## Verify users and shares

You can check samba users and available shares with following commands.

```bash
sudo pdbedit -L -v
net usershare info --long
```

## Configure firewall

If you are using ufw and by default you denny all traffic,
don't forget to allow access Samba service from the hosts in your local network. 

```bash
sudo ufw allow from 192.168.10.0/24 to any app samba
sudo ufw reload
```

## Other sharing options

Samba is not the only one service for file sharing. There are more or less advanced solutions which you can discover,
to name the few:

- NFS
- WebDav
- NextCloud
