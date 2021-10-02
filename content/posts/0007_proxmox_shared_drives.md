---
title: "Using a Samba share in Proxmox inside VMs"
date: 2021-09-16T21:05:27+01:00
tags: ["tech"]
draft: true
---

Proxmox Shared

Adding a mounted drive to Proxmox via Samba (Window's own file sharing) is my choice for the moment, until I get a NAS at least!

```bash
sudo apt-get install smbclient nfs-common cifs-utils
```

### Update firewall rules

Update `File and Printer Sharing (SMB-In)` with your subnet(s) to allow access to shares
If you also want to be able to ping across subnets, while you're here update `File and Printer Sharing (Echo Request ICMPv4-In)` with the same subnets.

Modify hosts file and add remote file handler (optional?)

Follow instructions in README to mount folder.

In proxmox:

Proxmox > Datacenter > Storage > Add > Directory

```
ID = <your name>
Directory= /mnt/myshare
```
