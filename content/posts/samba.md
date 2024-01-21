+++
date = 2024-01-21
publishDate = 2024-01-21
title = "Samba server setup on Ubuntu"
description = "How to quickly setup Samba server on Ubuntu"
tags = [
  "samba", "services", "networking", "admin"
]
categories = [
  "tools"
]
images = ["/images/samba.jpg"]
+++

{{< figure alt="Samba - carnival in Brasil" src="/images/samba.jpg" caption="Samba" >}}

## Install samba

```bash
sudo apt update
sudo apt install samba
```

## Add samba user for yourself

```bash
sudo smbpasswd -a $USER
sudo usermod -a -G sambashare $USER
net usershare add priv ~/Documents/priv "My Documents" $USER:f,everyone:d guest_ok=n
```

## Edit config

```bash
sudo nano /etc/samba/smb.conf
sudo service smbd restart
```

## Add accounts for others

```bash
sudo useradd -M -G sambashare karen
sudo passwd -l karen
net usershare add documents ~/Documents/shared "Shared Documents" $USER:f,karen:f,everyone:d guest_ok=n
```

## Verify users and shares

```bash
sudo pdbedit -L -v
net usershare info --long
```

## Configure firewall

```bash
sudo ufw allow from 192.168.10.0/24 to any app samba
sudo ufw reload
```
