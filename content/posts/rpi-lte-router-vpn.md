+++
date = 2023-11-17
publishDate = 2023-11-17
title = "VPN with nmcli on Raspberry Pi"
description = "Configuring Raspberry Pi networking with VPN"
tags = [
  "IoT",
  "networking",
  "VPN",
  "admin"
]
categories = [
  "hardware"
]
series = [
  "RaspberryPi"
]
images = ["/images/vpn.jpg"]
+++

In my [previous article](../rpi-lte-router) I described how to setup cellurar networking on RaspberryPi with GSM hat.
The problem is most often you will not get public IP address from your operator when using cellurar network.
It means difficulties to access to your RaspberryPi and it's network from remote location.
On the other hand the device is not exposed to the Internet threats.
 
In my case I want to use my RPi to control survilance system installed in a holiday cottage.
And I need to access RPi from my home in a downtown. To achive that I'm using OpenVPN tunnel over public internet.
VPN gives me both secure and easy access to the router.

In this article I will show how to enable VPN connection over cellurar connection configured before.
Again, I will give a recipe for NetworkManager and nmcli setup.
These tools will manage openvpn client connection for us.

{{< figure alt="Shield as a part of armor" src="/images/vpn.jpg" caption="Virtual Private Networking" >}}

## OVPN configuration file

A prerequisite to configure a VPN connection is to obtain a OpenVPN configuration file.
Typicaly you can get one from your OpenVPN server.

In my case I have Asus home router with built-in OpenVPN server. In this article I skip description how to 
configure OpenVPN server and generate PKI certificates. It depends on your software or equipment.

I will assume that OVPN config file contains server CA and client certificates together with password protected private key.

## Importing OpenVPN configuration

Make sure you have required OS packages:
```bash
sudo network-manager-openvpn
```

Import your OpenVPN configuration file to NetworkManager:
```bash
sudo nmcli connection import file rpi.ovpn
```

Example output:
```
Connection 'rpi' (24c3311d-c49f-4392-931d-7bd58cf8b4d3) successfully added.
```

Please note down the ID of your new 'rpi' vpn connection. We will use that in the next step.

## Additional setup

After importing connection, we can tweak several settings.

```shell
sudo nmcli connection modify rpi +vpn.user-name rpi
sudo nmcli connection modify rpi +vpn.password-flag 0
sudo nmcli connection modify rpi +vpn.cert-pass-flag 0
sudo nmcli connection modify rpi +vpn.secrets password='VPN_USER_PASSWORD'
sudo nmcli connection modify rpi +vpn.secrets cert-pass='PRIVATE_KEY_PASSWORD'
sudo nmcli connection modify t-mobile +connection.secondaries 24c3311d-c49f-4392-931d-7bd58cf8b4d3
```

`user-name` param is self explenatory, although `password-flag` and `cert-pass-flag` require some comments.
Normally, in a desktop environment NetworkManager will read passwords from password manager like Gnome keyring.
In automated, non-interactive setups it may make more sense to store passwords in NetworkManager connection config.
To accomplish that we set the flags to `0`.

Last line will set VPN connection as a secondary connection for a GSM connection, which means VPN client connection
will be started whenever the GSM connectionn is up.

## Web resources

* [nmcli man page](https://www.linux.org/docs/man1/nmcli.html)
* [OpenVPN Community docs](https://openvpn.net/community-resources/reference-manual-for-openvpn-2-4/)
