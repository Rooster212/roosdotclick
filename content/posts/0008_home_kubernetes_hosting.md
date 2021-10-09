---
title: "Hosting Services at Home with k3os and Terraform"
date: 2021-10-07T20:00:00+01:00
slug: "hosting-services-at-home-with-k3os-and-terraform"
tags: ["tech", "self hosting", "terraform", "kubernetes"]
draft: true
---

As I mentioned at the end of my previous post, there is a [great GitHub repo](https://github.com/lukaszraczylo/rpi-home-cluster-setup) that I found which covers a lot of the same bits and pieces I want to use (which I'll cover below).

Something my girlfriend uses for work that she pointed me at which is super useful is https://k8slens.dev/ - it allows you to visualise all the complicated stuff happening in your cluster.

## k3os setup

why k3os

install in proxmox

Need to get contents of `/etc/rancher/k3s/k3s.yaml` from the k3os machine to connect to the machine from another.

For me, I wanted to ssh onto box to copy the file. To SSH I had to use KVM to enable SSH first. For me it was easiest to allow Password authentication for SSH to logon. Credentials should be username `rancher` then whatever password you set

Edit `/etc/ssh/sshd_config` and change the line that says `PasswordAuthentication no` to `PasswordAuthentication yes`

then `sudo service sshd restart`

Then, you can run `scp root@192.168.0.100:/etc/rancher/k3s/k3s.yaml ~/my-destination-folder`, replacing the IP and the destination folder with what you want. In the file you'll want to place it somewhere you can use (in my case, I just replaced `~/.kube/config` with the one from the server). You'll even need

## Configuration

Firstly, I need a load balancer. k3s has a built in/default load balancer, however it doesn't appear to be that popular, so I'm going to disable it and run MetalLB.

There's a detailed explaination [on MetalLB's site](https://metallb.org/concepts/layer2/), but essentially it assigns IP addresses and allows for networking in a customiseable manner. It is quite technical. There are some caveats running it in k3s, namely that we need to disable the built in one. That is documented [on the MetalLB site here](https://metallb.org/configuration/k3s/).

Kubernetes networking is quite complex and I definitely don't fully understand it, so I'm not going to try and explain it...!