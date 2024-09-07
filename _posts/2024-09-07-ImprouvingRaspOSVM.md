---
title: How to improuve the Raspberry Pi OS image for a virtual machine
date: 2024-09-07 16:11:00 -0300
categories: [proxmox, raspberry pi]
tags: [proxmox, raspbian, raspos, homeserver, virtual machine]
---

## Introduction

### Why?

At the introduction of [my previous post]({% post_url 2024-09-02-CreatingRaspOSVM %}) I discussed some of the issues Raspberry Pi OS (RaspOS) has in regard of being used in a Virtual Machine (VM). The only real advantage of using such a system is the support one can get from the community. If you find a problem, someone, somewhere over the seas is there waiting to help you. The same is not true with Ubuntu for instance. A silly question put on a Ubuntu forum will be ignored or answered with a “buy a better equipment” or “use the proper software”. Well, we don’t have and neither wish to have any equipment more expensive than ours Raspberry Pis (RBP) nor can run another thing comfortably on the RBP.

Moreover, a RBP 4 or 5 is more than enough to run anything one may need in a homelab or homeserver. Actually, I managed to run all my stuff on a Lattepanda V1 plus a RBP 3B+. However, the resources one can have from such hardware are limited. Once committed to this kind of equipment, one must be wise on how to spend one’s GiBs and GFLOPS. 

In this post I show you how I managed to reduce the minimum space taken by a working image of RaspOS and what services can be safely stopped to reduce the impact of the idle activity of all VMs on the server.

### Things I assume you already know

I hope you have read all my previous posts, starting on [this one]({% post_url 2024-08-26-ProxmoxInstalationOnRaspberryPi %}), on this series and already know all the stuff I mentioned previously.

## Preparing the tools I needed

### A desktop

So far, I had two VMs on my Proxmox server, clones of each other, one fully configured and other stopped at the first boot. 
