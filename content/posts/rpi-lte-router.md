+++ 
date = 2023-11-04
publishDate = 2023-11-04
title = "LTE router on Raspberry Pi"
description = "Configuring cellurar networking with Waveshare GSM hat"
tags = [
  "IoT",
  "networking",
  "admin"
]
categories = [
  "hardware"
]
series = [
  "RaspberryPi"
]
images = ["/images/telephone.jpg"]
+++

I finally managed to find some free time to take a look on Waveshare SIM7600E-H 4G HAT.
I bought it to extend capabilities of my Raspberry Pi 4 model B single board computer.

Here, I want to give a short tutorial how to setup LTE WAN networking in Raspberry Pi SBC.
I will describe how to make use of the network tooling in new Raspberry Pi OS which is based on recent Debian Bookworm release.
At the end, I will present configuration of WAN connection sharing with other hosts in a LAN.

I struggled a bit with LTE modem configuration, but finally I managed to come up with the setup in three simple, fast steps.

{{< figure alt="Convoluted networking - telephone pole with number of wires" src="/images/telephone.jpg" caption="Cellular networking" >}}

## Hardware assembly

I didn't install Waveshare SIM7600E-H as a regular hat, but I used USB cable solely to connect the device with RPi board.
SIM7600E-H 4G HAT is supplied together with two USB cables. One of them is quite long, so you can position hat in some distance from RPi board.
Thanks to that you can put hat with antenna in convient place for better signal reception.

Ofcourse before start using the device, we also need to insert SIM card from your mobile operator 
and connect provided antenna to the "main" socket.

Quite important thing is to set UART JMP jumper in position A.

{{< figure alt="Raspberry Pi with connected Waveshare SIM7600E-H 4G HAT" src="/images/waveshare-sim7600e-h.jpg" caption="Assembled GSM kit" >}}

## OS installation

I chose official [Raspberry Pi OS imager](https://www.raspberrypi.com/software/) to install operation system on the board.
Since I am using RPi as a server I picked Lite OS version. When I purchased my board only 32-bit Raspbian OS was available.
Now I can fully leverage on 64-bit arm architecture of RPi 4B with Raspberry Pi OS.
At the beginning, I installed Ununtu arm64 which was released before 64-bit version of Raspberry Pi OS.

Before writing OS image to a memory card, I edited the settings to set a hostname and enable SSH server with public key login.
I had some difficulties with ED25519 key, so I stick to RSA. I haven't have chance yet to explore the issue further.

## Installing required packages

First, let's install neccessary tooling.

```shell
sudo apt install libqmi-utils udhcpc
```

QMI is a communication protocol for modems built on top of Qualcomm chips. Modem Manager can handle `libqmi` library calls for you.
Another tool - `udhcpc` is required to obtain WAN IP address from mobile operator via DHCP.

For seeting up a cellurar networking NetworkManager and ModemManager services will also be required in our setup.
Although, they should be installed and enabled by default in the newest RPi OS. You can verify it with:

```shell
sudo systemctl status NetworkManager
sudo systemctl status ModemManager
```

## Configuring modem connection and routing

Debian Bookworm is equiped with several interesting console tools for network setup:

* `nmtui` - NetworkManager terminal user interface
* `nmcli` - NetworkManager command line interface
* `mmcli` - ModemManager command line tool

Let's start network configuration with cellurar networking. In order to establish mobile connection check the commands below.

List available modems:
```shell
mmcli -L
```

Print details of the first modem:
```shell
mmcli -m 0
```

Unlock SIM card with PIN number (replace xxxx with your PIN): 
```shell
sudo mmcli -m 0 -i 0 --pin=xxxx
```

Enable modem:
```shell
sudo mmcli -m 0 -e
```

Start a new connection (please adhere you settings like APN name, IP type or authentication mode - check that with your mobile operator):
```shell
sudo mmcli -m 0 --simple-connect="apn=internet,ip-type=ipv4,allowed-auth=pap"
```

At this stage, the cellurar connection should be up. You can verify it with commands:
```shell
ip a
ping -I wwan0 8.8.8.8
```

As a next step we probably want to add a gsm connection profile in Network Manager:
```shell
sudo nmcli c add type gsm ifname '*' con-name 't-mobile' apn internet pin xxxx connection.autoconnect yes
```

Setting `connection.autoconnect` property will make the connection to start at the system boot.

The last, optional step is to enable routing, i.e. share the mobile connection with other hosts in the cable network.
It is quite simple. Just run `sudo nmtui` and find in termianl UI option to edit your wired connection. Switch mode from 'auto' to 'shared'.

By default 'shared' mode will set static ip `10.72.0.1/24` for `eth0` interface. It will also start DHCP server on that interface, 
enable NAT and IP masquarade. You can change default IP by setting desired IP in `address` field.

## Trobleshooting

You may find below commands useful in diagnosing connectivity problems.

```shell
lsusb
lsusb -t
ip a
ping -I 8.8.8.8
mmcli -L
mmcli -m 0
mmcli -m 0 -b 0
mmcli -m 0 -b 1
```

For troubleshooting your modem you may want to install `minicom` AT terminal app.
```shell
sudo apt install minicom
```

Then you can connect to modem TTY port and issue AT commands (device name may be different):
```shell
sudo minicom /dev/ttyUSB2
```

Some handy AT commands can be found on [Waveshare FAQ](https://www.waveshare.com/wiki/SIM7600E-H_4G_HAT#FAQ).
You can also find there a description of different modem operating modes. 

In my case I need to switch modem to RNDIS (9001) mode and change default APN name:

```
AT+CUSBPIDSWITCH=9001,1,1
AT+CGDCONT=1,"IP","internet","0.0.0.0",0,0
```

You can also use qmi directly and bypass ModemManager. Example diagnostic commands are:

```shell
sudo qmicli -d /dev/cdc-wdm0 -w
sudo qmicli -d /dev/cdc-wdm0 --dms-get-operating-mode
sudo qmicli -d /dev/cdc-wdm0 --nas-get-signal-strength
sudo qmicli -d /dev/cdc-wdm0 --nas-get-home-network
```

## Web resources

* [Waveshare SIM7600E-H 4G HAT wiki page](https://www.waveshare.com/wiki/SIM7600E-H_4G_HAT)
* [Raspberry Pi OS imager docs](https://www.raspberrypi.com/documentation/computers/getting-started.html#install-using-imager)
* [Post on the other blog re cellurar connectivity in RPi](https://www.wevolver.com/article/add-cellular-connectivity-to-your-raspberry-pi)
* [NetworkManager shared connection configuration](https://fedoramagazine.org/internet-connection-sharing-networkmanager/)
* [DNSmasq configuration](https://docs.fedoraproject.org/en-US/fedora-server/administration/dnsmasq/)
* [Waveshare SIM7600 overview (in polish)](https://mikrokontroler.pl/2020/05/05/nakladka-na-raspberry-pi-z-modemem-4g-waveshare-sim7600e-4g-hat/)
* [Manual QMI setup](https://www.embeddedpi.com/documentation/3g-4g-modems/raspberry-pi-sierra-wireless-mc7455-modem-raw-ip-qmi-interface-setup)
* [SIM7600X chip details](https://www.simcom.com/product/SIM7600X.html)
