---
title: "Attempting to Host Services in Docker Swarm"
date: 2021-10-02T16:55:00+01:00
slug: "attempted-hosting-in-docker-swarm"
tags: ["tech", "self hosting"]
draft: false
---

I tried to use Docker Swarm to expand my self-hosting capabilities. My thinking was that I have my current hosting in a `docker-compose.yml` file, so I should easily be able to use Swarm. Right?

I span up 3 VMs in Proxmox and added them to a swarm (relatively easy - I found a great tutorial that I half followed [here](https://codefresh.io/docker-tutorial/deploy-docker-compose-v3-swarm-mode-cluster/) so I'm not going to rehash it). I didn't follow it exactly (as the example was using Docker-in-Docker). I setup the swarm manually then wrote a script to add in a registry proxy (I almost immediately ran into the Docker Hub pull limit when experimenting with Swarm).

I used this script to setup a registry locally on the Swarm (as well as the swarm visualizer, which is a nice debugging tool)

```bash
#!/bin/bash -e

docker run -d --restart=always -p 4000:5000 \
    --name registry_through_cache \
    -v registry-mirror:/var/lib/registry \
    -e REGISTRY_PROXY_REMOTEURL=https://registry-1.docker.io \
    -e REGISTRY_PROXY_USERNAME=rooster212 \
    -e REGISTRY_PROXY_PASSWORD=my-access-key \
    registry:2.7

docker run -it -d  --restart=always \
    --name swarm_visualizer -p 8000:8080 -e HOST=localhost \
    -v /var/run/docker.sock:/var/run/docker.sock \
    dockersamples/visualizer
```

I then updated the Docker `daemon.json` file (`/etc/docker/daemon.json`) which also didn't exist on my nodes so I created it. The file looks like this on my first host (that is hosting the registry):
```json
{
  "registry-mirrors": ["http://localhost:4000"]
}
```
And on the other nodes,
```json
{
  "registry-mirrors": ["http://192.168.53.21:4000"]
}
```

Then I restarted Docker on all nodes by running `sudo systemctl restart docker` individually.

## Running a stack

I wrote a basic Docker Compose to host Apache Guacamole (which is awesome by the way)

```yaml
version: '3'

services:
  guacdb:
    hostname: guacdb
    image: mariadb:10.6
    environment:
      MARIADB_ROOT_PASSWORD: ${GUAC_DB_ROOT_PASSWORD}
      MARIADB_DATABASE: 'guacamole_db'
      MARIADB_USER: 'guacamole_user'
      MARIADB_PASSWORD: ${GUAC_DB_USER_PASSWORD}
    volumes:
      - 'guac-mariadb:/var/lib/mysql'
    deploy:
      placement:
        constraints: [node.role == manager]
    networks:
      guac1:
       aliases:
          - guacdb

  guacd:
    hostname: guacd
    image: guacamole/guacd
    deploy:
      placement:
        constraints: [node.role == manager]
    networks:
      guac1:
        aliases:
          - guacd

  guacamole:
    hostname: guacamole
    image: 'guacamole/guacamole:latest'
    deploy:
      placement:
        constraints: [node.role == manager]
    ports:
      - 8080:8080
    environment:
      GUACD_HOSTNAME: "guacd"
      MYSQL_HOSTNAME: "guacdb"
      MYSQL_DATABASE: "guacamole_db"
      MYSQL_USER: "guacamole_user"
      MYSQL_PASSWORD: ${GUAC_DB_USER_PASSWORD}
    depends_on:
      - guacdb
      - guacd
    networks:
      guac1:
        aliases:
          - guacamole
```

## The first problem - secrets

The first issue I ran into immediately was that getting secrets into my containers seems very difficult. Environment variables were my first go-to. However, it turns out environment variables are a bit of a nightmare. I actually couldn't get them working at all. I found [an open issue here](https://github.com/moby/moby/issues/29133#issuecomment-746528667) on Moby (Docker's open source core) which detailed some potential workarounds. I tried a few in a script but, I just _could not_ get it to work. I'm probably missing something obvious but I tried pretty much everything in the thread. My script in the end had the script in below, but it still didn't work. I am 99% sure I'm just missing something obvious here but, I deployed the stack and none of the variables I expected to be there were populated.

```
set -a
. ./.guac.env
docker stack deploy -c docker-compose.guac.yml guacamole
```

Of course, there is a [built in Swarm secrets mechanism](https://docs.docker.com/engine/swarm/secrets/) but it doesn't seem very mature compared to Kubernetes. You can't make the secrets appear as environment variables - they simply appear as a mounted file. I understand _why_ it doesn't (you could just run `docker inspect <container>` and get the secret value) but it feels like this needs solving by Docker. The secrets mechanism as is would have worked for my MariaDB container, as it supports the file path in lieu of environment variables but it would not have worked for the main Guacamole container.

## Problem 2 - Networking

I'm familiar with Docker networking having hosted my current services through Docker for ages, however to me it seems like there are differences that I don't yet understand. There are a bunch of topics on this - it looks like you have to address by service name rather than container name (which makes sense with scaling etc.) and then there might be a bug with IP addresses being 1 less in the last octet... maybe. Seems to be mixed issues on this one but I dug up a couple ([1](https://github.com/docker/for-linux/issues/897)).

## Problem 3 - Volumes

So volumes I haven't dug into that much. However, I did find some interesting stuff around hosting Samba shares (which I do need to do to access my storage on my server). I haven't attempted this but it seems like it should work. In an ideal world I'd like to declare everything in a more IaC way rather than CLI commands (like declaring in a Compose file) but this seems like it would work fine.

```bash
# https://docs.docker.com/storage/volumes/#create-cifssamba-volumes

docker volume create \
    --driver local \
    --opt type=cifs \
    --opt device=//uxxxxx.your-server.de/backup \
    --opt o=addr=uxxxxx.your-server.de,username=uxxxxxxx,password=*****,file_mode=0777,dir_mode=0777 \
    --name cif-volume
```

I have a few objectives with volumes - I want to be able to 'seed' the volume with data (a suggested workaround is to use `docker cp` on a container to copy in configuration at some point after deployment, which would work but again seems a bit less infrastructure as code). I also want to be able to backup volumes easily, as well as allowing them to move across nodes or be shared as necessary etc. Named volumes in Docker would solve most of these issues but I'm not 100% sure how easy backing up volume data is either.

Again, I think I could learn all of this... but I have another plan.

## Conclusion

In conclusion - I have been a _little_ burned with this. I think it's nice and easy to get setup with Swarm compared to say Kubernetes but I don't really know enough about the nuances. For me, I think I'm going to throw myself into the deep end and use [k3os](https://k3os.io/) and [Terraform](https://www.terraform.io/) to pull together my setup in a fully infrastructure as code way. [I found a great repo](https://github.com/lukaszraczylo/rpi-home-cluster-setup) that shows a fully automated Raspberry Pi k8s cluster with some of the services I'm looking to use (such as Prometheus and Grafana) defined in Terraform. So that'll be some great inspiration to help me get started.

I know people will ask, so here's a list of services I'm running in Docker right now that I want to replicate in a distributed way:

* [Traefik](https://traefik.io/) - Handles routing for HTTP, TCP and UDP services. I wrote a [previous article]({{< relref "0005_hosting_services_using_cloudflared.md" >}}) on how I use Cloudflare tunnel to expose those to the internet.
* [Apache Guacamole](https://guacamole.apache.org/) - basically VNC, RDP and SSH (and others) all in a browser. Great for handling multiple servers on my network.
* [Tautulli](https://tautulli.com/) - Monitors my Plex server usage in more detail than Plex does.
* [ownCloud](https://owncloud.com/) - Self hosted storage, kinda like Dropbox but has loads of extra functionality.
* [Calibre Web](https://github.com/janeczku/calibre-web) - A web app for e-books.
* [Bazarr](https://www.bazarr.media/) - Downloads subtitles automagically for my Plex media.
* [Jellyfin](https://jellyfin.org/) - An alternative media hosting server. I mainly just have this as a backup for Plex but it also is completely open source.
* [Motioneye](https://github.com/ccrisan/motioneye/) - A CCTV frontend, I have 2 cameras on my house right now with a couple more to install soon.
* [GitLab Runner](https://docs.gitlab.com/runner/) - I'm a big fan of GitLab and running your own runner is easy, and you can have a secure environment for your builds. I will hopefully use this to run my IaC straight back into my k3s cluster without having to worry about credentials or exposing the cluster to the internet.

Stay tuned - I'll almost certainly be giving Terraform a go soon.