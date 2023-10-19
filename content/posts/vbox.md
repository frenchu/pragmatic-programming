+++ 
date = 2023-10-18
publishDate = 2023-10-18
title = "Setting up networking between VirtualBox VMs"
description = ""
tags = [
    "virtualization",
    "networking"
]
categories = [
    "devops"
]
images = ["/images/boxes.jpg"]
+++

_Networking in VirtualBox_

{{< figure alt="Virtual Machines - wooden boxes on the ship" src="/images/boxes.jpg" caption="Photo by Pixabay on Pexels" >}}

There are several modes one can use for networking in VirtualBox. In this article I presented one way for network setup.
VMs talk each other using internal network separated from the host machine. Additionaly, I created OpnSense router to access VMs in internal network from the host.

To quickly spin desired number of VMs I used VBoxManage CLI utility and Ubuntu VBox cloud image.

## Configuration steps

### Install OpnSense

Follow guide in the [referenced article](https://techsphinx.com/hacking/install-opnsense-on-virtualbox/).

After starting OpnSense and configuring both WAN and LAN interfaces, enable web ui to be accessed from WAN (host network).

Follow the steps:

1. Start OpnSense VM
1. Enter OpnSense shell (option no. #8 in the menu)
1. Disable packet filtering `pfctl -d`
1. Create firewall rule for WAN interface and port 80 or 443

### Prepare cloud-init data

```shell
echo "instance-id: $(uuidgen)" > meta-data
echo "local-hostname: my-hostname" >> meta-data

cloud-localds seed.iso user-data meta-data
```

To prepare seed.iso image, first `user-data` config file has to be created. `user-data` typically contains information about user accounts to be created and/or packages to be installed. For more information check cloud-init docs.

By default cloud-init configures first available network interface in VM to obtain IP address from DHCP. OpnSense provides DHCP and DNS services. To distinguish VMs in the internal local network, change `my-hostname` in `meta-data` to unique name accross the internal network.

### Create VM from recent Ubuntu cloud image

```shell
VBoxManage import jammy-server-cloudimg-amd64.ova --vsys 0 --vmname my-vm --cpus 2 --memory 2048 --unit 9 --ignore
```

This command imports a new virtual machine from VirtualBox compatible image.
The OVA Ubuntu image is available to download from [Ubuntu cloud images](https://cloud-images.ubuntu.com/) website.

### Provide cloud-init configuration to VM

```shell
VBoxManage storageattach my-vm --storagectl IDE --port 0 --device 0 --type dvddrive --medium seed.iso
```

The could-init configuration is read from previously created ISO image mounted as DVD-ROM in VM.

### Attach VM to the internal network

```shell
VBoxManage modifyvm my-vm --nic1 intnet --intnet1 my-intnet --nictype1 virtio --nic-promisc1 allow-vms
```

VirtIO network interface is selected, which gives more performant network communication in virtualized environment.

## Summary

There are bash scripts available on my [github](https://github.com/frenchu/vbox-vm-setup), which can simplify the process of creating the VMs.

## References

* [OpnSense in VirtualBox VM](https://techsphinx.com/hacking/install-opnsense-on-virtualbox/)
* [Ubuntu cloud images](https://cloud-images.ubuntu.com/)
* [Sample shell script](https://github.com/frenchu/vbox-vm-setup)
