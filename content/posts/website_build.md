---
title: "Website Build - Part 1"
date: 2020-04-04T13:30:00Z
tags: ["tech","coding"]
draft: false
---

As we've been quarantined, I've taken the opportunity between working, eating and sleeping to finally get a website up and running. It's open source [on GitLab here](https://gitlab.com/Rooster212/roosdotclick), and I'll be looking to add to this over time.

I went with Hugo to create this site, as it generates a static HTML/CSS site, which I've always preferred for the performance, cacheability and general ease of management.

I host the site alongside many other projects on a server hosting Docker containers, with NGINX as the entrypoint behind CloudFlare for caching and DDOS protection.

One of the things I wanted to do with this site was to create a separate endpoint for asset hosting such as images and video. For video, solutions exist such as Vimeo Pro and other similar solutions, but I don't have any real need to do that as I will rarely need to put video on the site. For images, I wanted to share full resolution images but I didn't want to have them hosted by the same container as this site, to allow for separate caching settings through CloudFlare. My solution for this was [MinIO](https://min.io/) and some custom NGINX routing.

MinIO is an S3-compatible object store, which for me made it easy to get setup, as I use AWS services for work, and it has a nice web interface for easy upload from anywhere. I put my NGINX config to redirect `i.rooster212.com` and `i.roos.click` to the `/image` path of the S3 endpoint internally, which makes for a nice and clean URL for hosting the images and video on the website via CloudFlare. I can also use [Cyberduck](https://cyberduck.io/) on Windows for easy upload. For Mac, I'd highly recommend [Transmit](https://www.panic.com/transmit/) from Panic, which is an excellent S3 browser.
