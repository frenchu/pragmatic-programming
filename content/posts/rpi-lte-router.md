+++ 
draft = true
date = 2023-11-04
publishDate = 2023-11-04
title = "LTE router on Raspberry Pi"
description = "Configuring cellurar networking with Waveshare GSM hat"
tags = [
  "hardware",
  "networking"
]
categories = [
  "IoT"
]
series = [
  "RaspberryPi"
]
+++

I finally managed to find some free time to take a look on Waveshare GSM HAT.
I bought it to extend capabilities of my Raspberry Pi 4 model B.

Here, I want to present short tutorial how to setup LTE WAN networking in Raspberry Pi SBC.
I will describe how to make use of network tooling in new Raspian OS based on Debian Bookworm.

I struggled a bit with LTE modem configuration, but finally I managed to come up with the setup in three simple steps.

{{< figure alt="Convoluted networking - telephone pole with number of wires" src="/images/telephone.jpg" caption="Cellular networking" >}}

## Hardware assembly

I don't use Waveshare xxx as a regular HAT, but I use USB cable solely to connect the device with RPi.
Waveshare GSM HAT is supplied together with two USB cables. One of them is quite long, so you can position hat in some distance from RPi board.
Thanks to that you can put hat with antenna in convient place for better signal reception.


## OS installation

I used official Raspberry Pi OS imager. Since I am using RPi as a server I picked Lite OS version.
When I purchased my board only 32bit RPi OS was available. Now I can leverage on 64bit arm architecture of RPi 4B with 64-bit Raspbian.
At the beginning I installed Ununtu 64-bit.

While writing OS image to TF card I edited settings before hand to set hostname and enable SSH server with public key login.
I had some difficulties with ED25519 key, so I used RSA. I didn't have chance to explore the issue further.

## Installing required packages

```
sudo apt install libqmi-utils udhcpc
```

For seeting up a cellurar networking NetworkManager and ModemManager services will also be required.
Although, they should be installed and enabled by default in the newest RPi OS. You can verify it with:

```
sudo systemctl NetworkManager status
sudo systemctl ModemManager status
```

## Configuring modem connection and routing

Debian Bookworm is equiped with several interesting console tools for network setup:

* `nmtui` - NetworkManager terminal user interface
* `nmcli` - NetworkManager command line interface
* `mmcli` - ModemManager command line tool

Let's begin with cellurar networking. In order to establish mobile connection check the commands below.

List available modems:
```
mmcli -L
```

Print details of the first modem:
```
mmcli -m 0
```

Unlock SIM card with PIN number (replace xxxx with your PIN): 
```
sudo mmcli -m 0 -i 0 --pin=xxxx
```

Enable modem:
```
sudo mmcli -m 0 -e
```

Start a new connection (please adhere you settings like APN name, IP type or authentication mode - check that with your mobile operator):
```
sudo mmcli -m 0 --simple-connect="apn=internet,ip-type=ipv4,allowed-auth=pap"
```

At this stage, the cellurar connection should be up. You can verify it, for example with:
```
ip a
ping -I wwan0 8.8.8.8
```

As a next step we probably want to add a gsm connection profile in Network Manager:
```
sudo nmcli c add type gsm ifname '*' con-name 't-mobile' apn internet pin xxxx connection.autoconnect yes
```

Setting `connection.autoconnect` property will make the connection to start at the system boot.

The last, optional step is to enable routing, i.e. share the mobile connection with other hosts in the cable network.
It is quite simple. Just run `sudo nmtui` and find in termianl UI option to edit your wired connection. Switch mode from 'auto' to 'shared'.

By default 'shared' mode will set static ip `10.42.0.1` for `eth0` interface and start DHCP server on that interface.
You can change it by setting different IP in `address` field.

## Trobleshooting

```
lsusb
lsusb -t
ip a
ping -I 8.8.8.8
mmcli -L
mmcli -m 0
```

For troubleshooting your modem you may want to install `minicom`.
```
sudo apt install minicom
```

Then you can connect to modem TTY port and issue AT commands (device name may be different):
```
sudo minicom /dev/ttyUSB2
```

Some handy AT commands can be found on [Waveshare wiki]().

In my case I need to switch modem to DUPA mode and change default APN name:

## Web resources


