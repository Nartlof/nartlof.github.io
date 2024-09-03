---
title: How to create a Raspberry Pi OS virtual machine under Proxmox
date: 2024-09-02 14:14:00 -0300
categories: [proxmox, raspberry pi]
tags: [proxmox, raspbian, raspos, homeserver, virtual machines]
---

## Introduction

### Why it is not a good idea and why I did it anyway.

In [my previous post]({% post_url 2024-08-26-ProxmoxInstalationOnRaspberryPi %}) I described the process I used to install Proxmox on a Raspberry Pi 4 (RBP 4) with 4GiB of RAM. The whole reason one installs Proxmox is to create Virtual Machines (VM) and containers. There are a number of tutorials on the internet on how to install Debian, Ubuntu, Fedora and other Linux distributions on a VM, but I found scarce information on how to create one using Raspberry Pi OS (RaspOS). In other words, how to have a RBP inside a RBP?

Spoiler alert, it is possible because I did just that, however, RaspOS is an operating system developed to run on a very specific hardware under a very specific condition. It was not meant to be run in a VM. It is much easier and efficient to use Debian, for instance, than RaspOS.

When running VMs on a small hardware such as a RBP 4 or 5, the OS’s footprint and idle processing must be as small as possible. Each VM should have installed exclusively the application it is supposed to run and nothing else. In that sense, having a bare bone installation of Debian or Ubuntu is the way to go. This is not the case with RaspOS, which comes with heavy weight packages that most of the time just stay there taking disk space. 

But of course, the main reason one considers installing RaspOS in a VM is to stretch one’s ability of doing stuff with stuff not suited for the stuff one’s doing, such as installing Proxmox on a hardware it is not meant to run on. We choose to do these things and do the other things, not because they are easy, but because they are weird.

### Things I assume you already know

As always, this is not meant to be a copy-and-past guide for dummies on how to do something you should not be doing, rather a report written to my future self on how I did those things that are making my hair fall out because I can’t fix them. I based all this instructions on [this post](https://forum.proxmox.com/threads/how-to-convert-raspberry-pi-os-images-and-import-to-proxmox.146837/) by [primetechguides](https://forum.proxmox.com/members/primetechguides.238776/) So, before you follow along, make sure you tick this items:

* You know how to install and configure RaspOS under normal circumstances.
* You have watched this [amazing tutorial on Proxmox](https://www.youtube.com/watch?v=5j0Zb6x_hOk&list=PLT98CRl2KxKHnlbYhtABg6cF50bYa8Ulo) by [Learn Linux TV](https://www.youtube.com/@LearnLinuxTV)


## Downloading stuff