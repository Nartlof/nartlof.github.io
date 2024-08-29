---
title: How to install Proxmox on a Raspberry Pi SBC and put it to work
date: 2024-08-26 09:37:00 -0300
categories: [proxmox, raspberry pi]
tags: [proxmox, raspbian, raspos, homeserver]
---

## Introduction

### Why?

Having a home server to run applications like ad-blockers and media centers is a must have. The Raspberry Pi (RBP) single board computer (SBC) is a cost effective way for achieving that and, in many cases, it is possible to run several services on the same hardware.  Unfortunately, some of those applications run better if left alone on their own hardware or can not run together with some other software for some particular conflict, let us say, the port 80 for example. In cases like that one is forced to have a second or third piece of equipment to accomplish the desired goal.

When we were restricted to the RBP 3 or RBP 3+, it was not a big deal. They were cheap and not particularly powerful, so having several of them running simultaneously did not sound unreasonable. Moreover, having several machines, each one doing just one thing makes maintenance easier. 

This scenario changed with the advent of the Raspberry Pi 4 and 5, which could have up to 8GB of RAM, a more capable processor and USB 3 connectivity, also with a not so friendly price tag. Having a bunch of those running idle most of the time in our homelab is a waste. In this new context we must run our applications on containers and virtual machines that reside on this new and much more capable hardware. 

Proxmox is a well known and vastly documented software which has this exact function. It is a hypervisor. With it we can create virtual machines and containers to run applications in complete isolation from each other on the same hardware. It is powerful enough to even host itself or other hypervisores, allowing the creation of virtual machines inside virtual machines.

That is the path I decided to follow, but I soon discovered it is not a particularly well lit one. Although knowing it was possible to run Proxmox on a RBP, I struggled to find bits and pieces of information here and there on how to do it, while walking in the darkness and bumping into walls all the time. The knowledge available is sparse and spread out, without a proper body or context. This post is my attempt to clear and illuminate this approach, so those who come after may have a pleasant journey.

### Things I assume you already know

If you are reading this post with the intention of replicating what I have done, keep in mind that it is not a copy-and-paste tutorial, rather it is mostly a report I am writing to remind myself how to do this in case I have to do it again. That said, these are the things I will assume you know how to do.

* Create a bootable media for a RBP and make it work.
* Use the terminal window to give commands.
* Use nano editor to change files.
* Set a static IP address for a computer on the DHCP server of your router.
* Use SSH to access your computer remotely.


## The Hardware

I used in this guide a RPB 4 with 4GB of RAM as the server basis. It would be better to have used a RPB 5 with 8GB of RAM, or a RPB 4 with at least 8GB of RAM, however this was the hardware I had on hand. In any case, the only thing necessary to later upgrade the setup with a RBP 4 to a RBP 5 is to swap one for the other.

Despite the fact I started the whole process of installation with a SD card, it was not used for long, being replaced by a SSD drive straight away. Using a SSD as the main storage media for the server is mandatory; a SD card would not last long nor provide the necessary speed needed. Among several choices of external hard drives, I decided to use a 240GB SATA SSD with an USB to SATA adaptor. I got this particular model in the picture below.

![Sata to USB adaptor](/assets/images/ProxmoxInstalationOnRaspberryPi/SataToUsbCable.jpg)

In summary, the hardware I was dealing with was:
* Raspberry Pi 4 with 4GB of RAM
* SD card large enough to hold a full installation of Raspberry Pi OS
* SATA SSD with 240GB of capacity
* USB 3.0 to SATA adaptor

## Preparing the backgroung

### Creating a bootable media

That was straightforward. I just open the latest version of Raspberry Pi Imager on my computer and copy the full version image with a desktop into the SD card. 

![Writing Raspberry Pi OS on the SD card](/assets/images/ProxmoxInstalationOnRaspberryPi/ImagerFirstInstall.jpg)

If you already have an installed version of Raspberry Pi OS, this step is superfluous. I just did it because all my RBPs are in use for other purposes and I don't want to disturb them by installing software I am going to use just once. Moreover, there is one problem with the RBP 4 drivers that must be addressed before I install anything on the SSH. As I already solved it on my other RBPs, I needed this fresh installation to show it.

Once I had the GUI in front of me, I started the process by opening a terminal window and typing the ubiquitous:

```bash
sudo apt update && sudo apt -y full-upgrade
```

The next thing I did was to install GParted and mtools. This was needed to create and rearrange some partitions on the SSD and made it more suitable for the hyperviser.

```bash
sudo apt -y install gparted mtools
```

### Solving some driver issues

The next thing to do was to write an image to the SSD and at this point my struggle began. I have installed Raspberry Pi OS on hard drives and SSDs several times using pen drives, external hard discs or SATA discs with USB adaptors with zero issues. However, I could not create a bootable SSD from my RBP 4. It failed each time at different moments. Sometimes during the writing, some during the verification, going well just ones only to fail during boot. I tried a number of times, some using the USB 3 ports and some the USB 2 ports. I also tried a different adaptor with no success. It worked for a time, but randomly failed, crashing the system in the process.

The most strange thing about this problem is that the very same SSD with the SATA to USB adaptor worked fine on a RBP 3B and RBP 3B+. I did the complete process of installing the system using a RPB 3B+ and it was a breeze. I was about to conclude that my RBP 4 had some manufacturing defect when I found [this article](https://smitchell.github.io/how-to-bind-to-the-right-usb-storage-driver) while researching another problem.

If I attached the SSD to a RBP 3 and run the command:

```bash
lsusb -t
```

I had the following result:

![Result of the lsusb command on a RBP 3](/assets/images/ProxmoxInstalationOnRaspberryPi/lsusb--t-rbp3.jpg)

The SSD was identified as a mass storage device and the _usb-storage_  driver was attached to it. So far, so good. That is the expected behavior. On the other hand, if I gave the same command to the RBP 4 the result was different.

![Result of the lsusb command on a RBP 4](/assets/images/ProxmoxInstalationOnRaspberryPi/lsusb--t-rbp4.jpg)

Now the driver was _uas_. The SSD is identified as a mass storage device and the ‘usb-storage’ driver is attached to it. So far, so good. That is the expected behavior. On the other hand, if I gave the same command to the RBP 4 the result was different.

I am convinced that it is a hardware related problem or something with the USB bus driver of the RBP 4 and 5. The same SD card inserted on a RBP 3 gives the correct driver. Fortunately, the solution was quite easy. I just had to tell the RBP to use the correct driver, but first I had to discover which one it was. For that I use the command ```lsusb``` again, but this time without any parameters.

```bash
lsusb
```

And there it was on the first line.

![ID of the device 2 on bus 2](/assets/images/ProxmoxInstalationOnRaspberryPi/lsusb-rbp4.jpg)

Then I edited the file cmdline.txt to add the command ```usb-storage.quirks=152d:0578:u``` at the beginning.

```bash
sudo nano /boot/firmware/cmdline.txt
```

My cmdline.txt ended up looking like this:

```text
usb-storage.quirks=152d:0578:u console=serial0,115200 console=tty1 root=PARTUUID=85f2652a-02 rootfstype=ext4 fsck.repair=yes rootwait quiet splash plymouth.ignore-serial-consoles cfg80211.ieee80211_regdom=BR
```

Then I saved the file and rebooted the system.

```bash
sudo reboot
```

After the system was loaded again I repeated the ```lsusb -t``` command with the expected result.

![Result of the lsusb command on a RBP 4 after the usb-storage.quirks command](/assets/images/ProxmoxInstalationOnRaspberryPi/lsusb--t-rbp4B.jpg)

Be aware that you may not need this correction at all. If you run the command ```lsusb -t``` and the driver is _usb-storage_, just ignore all I have done. If not, replace the ID, that in my case was **152d:0578**, with the one you find for your own device, that will be, most likely, a different one.

### Creating a bootable SSD

Having now a reliably working RBP 4 I opened again the imager, this time on the Pi itself, and wrote the Lite version of Raspberry Pi OS on the SSD.

![Writing Raspberry Pi OS on the SSD](/assets/images/ProxmoxInstalationOnRaspberryPi/ImagerSecondInstall.jpg)

Once the process was finished, I opened a terminal window and mounted the boot partition on /mnt.

```bash
sudo mount /dev/sda1 /mnt 
```

I had to adjust the cmdlint.txt file on the SSD or it would fail at boot due to the wrong driver. This time, in order to edit the file, I typed:

```bash
sudo nano /mnt/cmdline.txt
```

And again made the same modification, including ```usb-storage.quirks=152d:0578:u``` at the beginning of the file, just like before, then saved and exited. It was time to ```umount``` that partition.

```bash
sudo umount /mnt
```

Then, I went back to the GUI for some deep dive into this new drive. I opened GParted, which I had installed previously, to make some modifications on the structure of the system. GParted asks for the user password to be opened. This is a kind of program that can cause a lot of damage if not used carefully and that is another good reason to create a new SD card with the only objective of creating this SSD. Fist I selected the correct drive I was going to edit, which was */dev/sda*.

![Selecting the partition on GParted](/assets/images/ProxmoxInstalationOnRaspberryPi/Gparted01.jpg)

The first partition on the SSD is the boot one. It is 512MiB long but just a fraction of it is used. I am a person from the time when my whole computer had just 640k of RAM and the hard drive was only 30MiB long. The entire operating system with most applications could fit in a floppy disc of just 360k. Having a partition of half a gigabyte to use less than 40MiB of it is offensive to me. So, I adjusted it to a more appropriate size. I right clicked on the partition and chose Resize/Move.

![Start the resize operation](/assets/images/ProxmoxInstalationOnRaspberryPi/Gparted02.jpg)

![Finishing the resize operation](/assets/images/ProxmoxInstalationOnRaspberryPi/Gparted03.jpg)

On the new windows that opened I changed the size to 96MiB, which is still an overkill but frees more than 80% of the original space. Next I clicked on Resize/Move. It was time to put to use the space conquered. I right clicked on the second partition and again chose Resize/Move. This time I altered the space before the partition from 416MiB to zero and clicked Resize/Move.

![Moving the second partition to the left](/assets/images/ProxmoxInstalationOnRaspberryPi/Gparted04.jpg)

As soon as I clicked Resize/Move I was greeted with this amicable warning.

![Warning from GParted](/assets/images/ProxmoxInstalationOnRaspberryPi/Gparted05.jpg)

It was just GParted being silly. I was not moving a boot partition. The boot partition is the one I resized and this one is not a Windows partition. I knew I had nothing to worry about and just clicked OK.

Following that, I focused my attention on the unallocated space at the end of the drive. I right clicked on it and chose New. First I changed the file system to linux-swap. On the New size box I wrote 4096MiB and used the mouse to drag it all the way to the end of the space.

![Adding a swap partition](/assets/images/ProxmoxInstalationOnRaspberryPi/Gparted06.jpg)

Choosing the size of a swap partition is not an exact science. What I do is to set the size of the partition to the same size of the installed memory. That is where I compromise. Some people say it is too much, some say it is too little. If you think I should use a different amount, feel free to do it your way.

Finely, I clicked on Apply All Operations, which is the little button with a green tick mark on it, to let GParted do its thing. I was again greeted with another silly message from GParted, being once more overprotective.

![Second warning from GParted](/assets/images/ProxmoxInstalationOnRaspberryPi/Gparted07.jpg)

I did not have nor would ever have a Windows system coexisting with Raspberry Pi OS, so this is another warning that was safe to ignore and click OK. The process finally started and soon enough it was concluded. 

![Applying changes](/assets/images/ProxmoxInstalationOnRaspberryPi/Gparted08.jpg)

The very last thing I did was to right click on the new swap partition and change its label to swap. Then I clicked OK and applied the changes once more.

![Changing the label of the new partition](/assets/images/ProxmoxInstalationOnRaspberryPi/Gparted09.jpg)

All being done, I shot the system down, removed the SD card and powered it again for the first boot from my new drive.