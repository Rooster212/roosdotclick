---
title: "Frigate NVR in Proxmox"
date: 2021-11-18T19:30:00+01:00
slug: "frigate-nvr-in-proxmox"
tags: ["tech", "self hosting"]
draft: true
---

# Get the LXC

https://pve.proxmox.com/wiki/Linux_Container#pct_settings

https://github.com/blakeblackshear/frigate/discussions/1111

```bash
pveam update
```


```bash
pveam available --section system
```

I picked the latest Debian 11 standard LXC image
```bash
pveam download local debian-11-standard_11.0-1_amd64.tar.gz
```

```
pveam list local
```


Update the .conf for the LXC before booting.

https://github.com/blakeblackshear/frigate/discussions/1111
https://forum.proxmox.com/threads/cant-start-lxc-conrtainer-with-lxc-mount-auto-after-upgrading-to-7.92429/


Append the following lines after the existing ones:

```conf
features: nesting=1
onboot: 1
lxc.cgroup2.devices.allow: c 226:0 rwm
lxc.cgroup2.devices.allow: c 226:128 rwm
lxc.cgroup2.devices.allow: c 29:0 rwm
lxc.cgroup2.devices.allow: c 189:* rwm
lxc.apparmor.profile: unconfined
lxc.cgroup2.devices.allow: a
lxc.mount.entry: /dev/dri/renderD128 dev/dri/renderD128 none bind,optional,create=file 0, 0
lxc.mount.entry: /dev/bus/usb/002/ dev/bus/usb/002/ none bind,optional,create=dir 0, 0
lxc.cap.drop:
```

Run commands one at a time
```
apt update

apt upgrade

apt install docker.io

systemctl enable docker

systemctl start docker

systemctl status docker
```

## Install docker-compose
https://docs.docker.com/compose/install/

## Add CIFS share
I'm hosting my file share on a Windows server

// TODO add credentials file and use in next step

Add a credentials file for mounting (as I'm running as root I'm just putting it in my `/root` folder)
```
touch .smbcredentials
chmod 600 ./.smbcredentials
```

Populate the file with the CIFS share username and password:
```
username=username
password=password
```

I added this to my `/etc/fstab` file:
```
//192.168.53.3/cctv /mnt/cctv cifs credentials=/root/.smbcredentials 0 0
```

