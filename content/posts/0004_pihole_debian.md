---
title: "Installing Pihole on Debian 11/Bullseye"
date: 2021-08-18T07:35:47+01:00
slug: "pihole-on-debian-11"
tags: ["tech", "pihole"]
draft: false
---

A quick article on how I got Pihole working on Debian 11 AKA Bullseye. It's not an officially supported OS yet, but I wanted to try getting it working.

It took a bit of digging to work out some of the weird cases, but after debugging the issues I've got it working.

Sources
* https://unix.stackexchange.com/questions/573152/how-to-troubleshoot-lighttpd-service-not-starting-up
* https://github.com/pi-hole/pi-hole/issues/1401
* https://discourse.pi-hole.net/t/pi-hole-on-debian-bullseye/43464/8

```bash
# install packages that we seem to need
sudo apt-get install wget curl net-tools gamin lighttpd lighttpd-mod-deflate

# Install Pihole. Piping to bash is controversial so download the file 
# if you feel the need to review the script before running.
curl -sSL https://install.pi-hole.net | PIHOLE_SKIP_OS_CHECK=true sudo -E bash

# How to change the admin password (I like to do this immediately)
PIHOLE_SKIP_OS_CHECK=true sudo -E pihole -a -p
```

Some of my debugging steps included needing to restart or see the status of services, so if you run into issues these commands may help you debug:
```bash
# To enable services needed by Pihole (may or may not be needed)
sudo systemctl enable pihole-FTL
sudo systemctl enable lighttpd

# To repair Pihole on an unsupported OS
PIHOLE_SKIP_OS_CHECK=true sudo -E pihole -r
```