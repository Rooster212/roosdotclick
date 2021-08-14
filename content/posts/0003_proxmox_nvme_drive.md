---
title: "Self Hosting Journey using Proxmox Part 2 - Extra Storage"
date: 2021-08-14T13:45:09+01:00
tags: ["tech", "self hosting", "proxmox"]
slug: 'proxmox-part-2-storage'
draft: false
---

The next step for me getting started with Proxmox was to add a bit more storage. The bundled 256GB SSDs are great for the basics, but I'm mainly concerned with the longevity of them (considering they are already well used!) and generally the fact that it's not really that much space.

Amazon to the rescue with some 1TB NVMe drives on sale for £59 which is an absolute bargain. I had to go with NVMe drives as it was the only spare slot for storage available in the miniature nodes I picked up (covered in my [last article]({{< relref "0002_vlans_and_proxmox.md" >}})).

Setting them up in Proxmox isn't just plug and play - we need to format the disks so that they will work. I'm sure there are several ways to do this but this is how I've done it.

Firstly, I created an LVM through the UI. This is super simple:

{{< figure src="/content/self_hosting/proxmox_vm_network.png" caption="VLANs" >}}

But I was only able to apply Disk images and containers to this. Whilst this is probably fine - I'd like to have the flexibility to store other things there (backups, ISO images etc.)

My NVMe devices in the Linux filesystem are `/dev/nvme0n1`.

```bash
fdisk /dev/nvme0n1
```