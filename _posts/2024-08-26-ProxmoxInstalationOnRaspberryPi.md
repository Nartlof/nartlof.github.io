---
title: How to install Proxmox on a Raspberry Pi SBC and put it to work
date: 2024-08-26 09:37:00 -0300
categories: [proxmox, raspberry pi]
tags: [proxmox, raspbian, raspos, homeserver]
---

# Introduction

Having a home server to run applications like ad-blockers and media centers is a must have. The Raspberry Pi (RBP) single board computer (SBC) is a cost effective way for achieving that and, in many cases, it is possible to run several services on the same hardware.  Unfortunately, some of those applications run better if left alone on their own hardware or can not run together with some other software for some particular conflict, let us say, the port 80 for example. In cases like that one is forced to have a second or third piece of equipment to accomplish the desired goal.

When we were restricted to the RBP 3 or RBP 3+, it was not a big deal. They were cheap and not particularly powerful, so having several of them running simultaneously did not sound unreasonable. Moreover, having several machines, each one doing just one thing makes maintenance easier. 

This scenario changed with the advent of the Raspberry Pi 4 and 5, which could have up to 8GB of RAM, a more capable processor and USB 3 connectivity, also with a not so friendly price tag. Having a bunch of those running idle most of the time in our homelab is a waste. In this new context we must run our applications on containers and virtual machines that reside on this new and much more capable hardware. 

Proxmox is a well known and vastelly documented software which has this exact function. It is a hypervisor. With it we can create virtual machines and containers to run applications in complete isolation from each other on the same hardware. It is powerful enough to even host itself or other hypervisor, allowing the creation of virtual machines inside virtual machines.

That is the path I decided to follow, but I soon discovered it is not a particularly well lightened one. Although knowing it was possible to run Proxmox on a RBP, I struggled to find bits and pieces of information here and there on how to do it, while walking in the darkness and bumping into walls all the time. The knowledge available is sparse and spreaded out, without a proper body or context. This post is my attempt to clear and illuminate this approach, so those who come after may have a pleasant journey.

# The Hardware

I used on this guide a RPB 4 with 4GB of RAM as the server basis. It would be better to have used a RPB 5 with 8GB of RAM, or a RPB 4 with at least 8GB of RAM, however this was the hardware I had on hand now. In any case, the only thing necessary to later upgrade the setup with a RBP 4 to a RBP 5 is to swap one for the other.

Despite the fact I started the whole process of installation with a SD card, it was not used for long, being replaced by a SSD drive soon later. Using a SSD as the main storage media for the server is mandatory. A SD card would not last long nor provide the necessary speed needed. Among several choices of external hard driver, I decided to use a 240GB SATA SSD with an USB to SATA adaptor. I got this particular model in the picture below.

![Sata to USB adaptor](/assets/images/ProxmoxInstalationOnRaspberryPi/SataToUsbCable.jpg)

In summary, the hardware I was dealing with was:
* Raspberry Pi 4 with 4GB of RAM
* SD card large enough to hold a full installation of Raspberry Pi OS
* SATA SSD with 240GB of capacity
* USB 3.0 to SATA adaptor

# Preparing the backgroung

That was straightforward. I just open the latest version of Raspberry Pi Imager on my computer and copy the full version image with a desktop into the SD card. Once I had it up and running started the process by the ubiquitous:

```bash
sudo apt update && sudo apt -y full-upgrade
```
The next thing I did was to install GParted and mtools. This was needed to create and rearrange some partitions on the SSD and made it more suitable for the hyperviser.

```bash
sudo apt -y install gparted mtools
```

