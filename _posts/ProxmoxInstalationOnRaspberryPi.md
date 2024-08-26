---
title: How to install Proxmox on a Raspberry Pi SBC and put it to work
date: 2024-08-26 9:37:00 -300
categories: [proxmox, raspberry pi]
tags: [proxmox, raspbian, raspos, homeserver]
---

# Introduction

Having a home server to run applications like ad-blockers and media centers is a must have. The Raspberry Pi (RBP) single board computer (SBC) is a cost effective way for achieving that and, in many cases, it is possible to run several services on the same hardware.  Unfortunately, some of those applications run better if left alone on their own hardware or can not run together with some other software for some particular conflict, let us say, the port 80 for example. In cases like that one is forced to have a second or third piece of equipment to accomplish the desired goal.

When we were restricted to the RBP 3 or RBP 3+, it was not a big deal. They were cheap and not particularly powerful, so having several of them running simultaneously did not sound unreasonable. Moreover, having several machines, each one doing just one thing makes maintenance easier. 

This scenario changed with the advent of the Raspberry Pi 4 and 5, which could have up to 8GB of RAM, a more capable processor and USB 3 connectivity, also with a not so friendly price tag. Having a bunch of those running idle most of the time in our homelab is a waste. In this new context we must run our applications on containers and virtual machines that reside on this new and much more capable hardware. 

Proxmox is a well known and vastelly documented software which has this exact function. It is a hypervisor. With it we can create virtual machines and containers to run applications in complete isolation from each other on the same hardware. It is powerful enough to even host itself or other hypervisor, allowing the creation of virtual machines inside virtual machines.

That is the path I decided to follow, but I soon discovered it is not a particularly well lightened one. Although knowing it was possible to run Proxmox on a RBP, I struggled to find bits and pieces of information here and there on how to do it, while walking in the darkness and bumping into walls all the time. The knowledge available is sparse and spreaded out, without a proper body or context. This post is my attempt to clear and illuminate this approach, so those who come after may have a pleasant journey.



![Sata to USB adaptor](/assets/images/ProxmoxInstalationOnRaspberryPi/SataToUsbCable.jpg)

