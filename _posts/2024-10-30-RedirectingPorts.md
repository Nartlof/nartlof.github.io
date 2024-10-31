---
title: Redirecting ports on Raspberry Pi OS
date: 2024-10-30 10:04:00 -0300
categories: [quick note, raspberry pi, debian]
tags: [iptables, redirection, linux, raspbian]
---

## Introduction

### Why

Some packages such as *Pi-hole* or *Jellyfin* have their web interface tied to some unusual port, as 8086 or 9090, instead of the standard 80 or 443. I think it is more convenient to have those web interfaces on standard ports. Sometimes it is possible to change it on the software configuration file, but it is different for each one of them. A much easier approach is to use ```iptables``` to redirect the traffic from any standard port to the one used by the program.

## The process

### Installing iptables

The first thing to do is to install ```iptables``` on your system.

```bash
sudo apt update && sudo apt -y upgrade && sudo apt install iptables
```

### Creating your rule

Let's say we have an application listening to port 9091 and we what it on port 80; we just need to give the command:

```bash
sudo iptables -A INPUT -i eth0 -p tcp --dport 80 -j ACCEPT
sudo iptables -A INPUT -i eth0 -p tcp --dport 9091 -j ACCEPT
sudo iptables -A PREROUTING -t nat -i eth0 -p tcp --dport 80 -j REDIRECT --to-port 9091
```

And, for IPv6

```bash
sudo ip6tables -A INPUT -i eth0 -p tcp --dport 80 -j ACCEPT
sudo ip6tables -A INPUT -i eth0 -p tcp --dport 9091 -j ACCEPT
sudo ip6tables -A PREROUTING -t nat -i eth0 -p tcp --dport 80 -j REDIRECT --to-port 9091
```

And that is it. The application can now be reached from port 80 and also 9091. The main advantage of this method is to keep the default port opened.

### Making it permanent

Unfortunately this rule will be lost as soon as the system is booted. To make it permanent the package ```iptables-persistent``` can be used.

```bash
sudo apt -y install iptables-persistent
```

During installation it will ask if the present rules should be saved, so it is a good idea install ```iptables``` first, make all rules, test them and then install ```iptables-persistent```

### Modifying the rules

The rules are saved on ```/etc/iptables/rules.v4``` and ```/etc/iptables/rules.v6```. One can edit those files but a better approach is just create any new rule you want with ```iptables``` command and then saving them with:

```bash
sudo su -c 'iptables-save > /etc/iptables/rules.v4'
```

or, in case of IPv6 rules:

```bash
sudo su -c 'ip6tables-save > /etc/iptables/rules.v6'
```

