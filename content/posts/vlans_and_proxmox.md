---
title: "Self Hosting Journey using Proxmox Part 1 - Starting Off"
date: 2021-08-08T22:37:49+01:00
tags: ["tech", "self hosting", "docker"]
draft: true
---

So I've been wanting to expand my home server hosting capabilities. I've got a load of things I host at home. Just a few of them are:
* I run the controller for my home SDN (software defined network), which probably warrants a post of it's own, in Docker! I personally run [TP-Link Omada](https://www.tp-link.com/uk/omada-sdn/) hardware.
* I host a [GitLab](https://about.gitlab.com/) runner, to allow me to run CI for my jobs in GitLab for free!
* I run [Owncloud](https://owncloud.com/) so I can have my own 'Dropbox' at home (with terabytes of storage!)
* [Tautulli](https://tautulli.com/) which is a monitor for my Plex server.

So to that end... I bought a job lot of mini PCs off eBay. I was partially inspired by the ServeTheHome Project TinyMiniMicro series (one of their excellent articles is [here](https://www.servethehome.com/lenovo-thinkcentre-m720q-tinyminimicro-feature/)), as well as some YouTubers experience with Raspberry Pi clusters. I thought about picking up one of the Raspberry Pi racks available (for 4-5 Pis) but at the end of the day, I feel I need more power than is available in your standard Raspberry Pi.

A few reasons for this choice really. I feel SD cards don't really cut the mustard when it comes to self hosting a lot of my services. I know you can run SSDs or flash drives on a Pi but, once you factor in everything needed to run something like that, the overall cost is quite high. The only thing that did appeal was using a load of [PoE+ HATs with Pis](https://thepihut.com/products/raspberry-pi-poe-plus-hat) (as I have a powerful PoE+ switch!) but the price just adds up for multiple nodes.

For me, I purchased a job lot of 5 HP Elitedesk 800 G2 Mini computers. These have clearly been pulled from an office or something like that, but they are in decent shape and came with 256GB SSDs (from the SMART data they are quite heavily used, but that's OK) and 8GB of RAM each, as well as Core i3 6100T CPUs. These are not super powerful by any means, but this does mean that these servers will sip a lot less power than the average PC. The power brick is rated at 35W but I highly doubt these will use anywhere near that under normal usage.

# Initial machines setup

My machines have come with pretty old BIOS images. As they are business machines, they actually come with a network BIOS updater (which is pretty awesome). I decided to jump straight to latest. They did a reboot and some scary beeping, but it came back up with no issues after a couple of minutes. The other thing is that I needed to connect via VGA to be able to use the BIOS and Proxmox installer. A bit of an inconvenience, but considering I have one and once Proxmox is installed I won't have to touch it again - I've got no problem. If I needed to I could get an active VGA to HDMI adapter to connect it to a more modern machine (or a KVM like [Pi-KVM](https://pikvm.org/), which I have my eye on when they are available!)

# Proxmox

Proxmox is super easy to install. I basically downloaded the ISO from the website, used balenaEtcher to write that to a USB stick then installed it on each machine. I've also followed [Networkchuck's excellent Proxmox starter video](https://www.youtube.com/watch?v=_u8qTN3cCnQ) as well as [Craft Computing's](https://www.youtube.com/watch?v=08b9DDJ_yf4) video on this subject which helped me a lot to get stuff sorted, as well as getting Proxmox.

Why Proxmox?
* Central management! All browser based and 'datacenter' level management. (Over ESXi, which I have previous experience with, which doesn't have free clustering)
* Keeping things up and running! This is especially important for my Omada controller and CCTV host.
* Flexiblity - I like the ability to create VMs. My current setup is a few VMs on a Windows machine under HyperV, but it's not a 'server' at the end of the day and has the usual Windows 10 issues like rebooting at random times. Use Linux for servers, always...!

Once you are installed, it's easy enough to get sorted.

## Datacenter setup

_TODO_

## Storage

Click on Datacenter and REMOVE the `local-lvm` under the Storage tab.

Run these in the shell via the Proxmox console on your node or nodes BEFORE you run any VMs (you'll probably mess something up if you don't do it before you make VMs)
```
lvremove /dev/pve/data
lvresize -l +100%FREE /dev/pve/root
resize2fs /dev/mapper/pve-root
```