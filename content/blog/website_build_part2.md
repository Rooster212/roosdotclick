---
title: "Website build - Part 2"
date: 2020-04-26T12:30:30+01:00
tags: ["tech","coding","blog"]
draft: false
---

[In my last post]({{< ref "website_build.md" >}}) I detailed a bit about how I built this site, and how I was using NGINX to host this on my server.

Since then, I've discovered the magic of [Traefik](https://docs.traefik.io/). Traefik allows for all the same things that NGINX does for my use case, but it allows for autodiscovery of configuration through Docker labels, which for me works really well as I host this site in Docker on a server. It also automatically configures LetsEncrypt, which for me is important - one of my domains is a `.dev` which requires HTTPS, and I feel in this day and age that HTTPS should really be mandatory.

I also had a play with Telegraf/InfluxDB/Grafana but I didn't get any decent results for now I've removed that from my configuration.

I spent a lot of time getting Traefik to work well, especially on my home server, as I host a lot of things at home (mainly for Plex and accessing my data remotely.)

When I was getting LetsEncrypt working, I ran into issues getting my DNS challenge working. I use CloudFlare for my DNS (and for the caching/DDoS/IP obfuscation) but I had a lot of issues. Some of those issues I believe were because [CloudFlare had some downtime](https://blog.cloudflare.com/cloudflare-dashboard-and-api-outage-on-april-15-2020/) while I was playing with it. However, getting it working in the end was actually super simple.

These command flags and environment variables go on the Traefik container itself - I'm using values from the `.env` file to hide the secrets from public view.

```yaml
  command:
    # ... more commands above
    - --certificatesResolvers.cloudflare-dns-resolver.acme.email=jamie@roos.click
    - --certificatesResolvers.cloudflare-dns-resolver.acme.storage=acme.json
    # Default: "https://acme-v02.api.letsencrypt.org/directory"
    - --certificatesResolvers.cloudflare-dns-resolver.acme.caServer=https://acme-staging-v02.api.letsencrypt.org/directory
    - --certificatesResolvers.cloudflare-dns-resolver.acme.keyType=RSA4096
    - --certificatesResolvers.cloudflare-dns-resolver.acme.dnsChallenge=true
    - --certificatesResolvers.cloudflare-dns-resolver.acme.dnsChallenge.provider=cloudflare
  environment:
    CF_API_EMAIL: jamieroos1@gmail.com
    CF_DNS_API_TOKEN: ${MY_CF_DNS_API_TOKEN}
    CF_ZONE_API_TOKEN: ${MY_CF_DNS_API_TOKEN}
```

Traefik works with labels (you can use files as well, but this works well for me). In this case, I've created a certificate resolver called `cloudflare-dns-resolver`. This name will be needed for setting up on your other containers to setup HTTPS.

Initally when setting it up, I recommend adding `--log.level=debug` and definitely using the staging endpoint from LetsEncrypt. They have strict usage limits. Traefik is quite talkative at Debug level but it helped me diagnose some weird issues (mainly that I was getting errors from CloudFlare due to DNS records not existing, which I think was [down to the downtime they had at the time](https://blog.cloudflare.com/cloudflare-dashboard-and-api-outage-on-april-15-2020/)).

On the container you want to have an HTTPS endpoint and use your DNS resolver, you then need to specify some other labels. For example, for this site my configuration looks like the below in my `docker-compose.yml`. I've commented it to help guide you if you're struggling.

```yaml
  roosdotclick:
    image: registry.gitlab.com/rooster212/roosdotclick/roosdotclick
    restart: unless-stopped
    networks:
    - internal
    labels:
    # Enable Traefik for this container - I explicitly disable auto expose in the Traefik commands.
    - "traefik.enable=true"

    # Setup HTTP -> HTTPS redirect
    - "traefik.http.routers.roosdotclick-insecure.entrypoints=web"
    - "traefik.http.routers.roosdotclick-insecure.rule=Host(`roos.click`)"
    - "traefik.http.middlewares.roosdotclick-insecure-redirect.redirectscheme.scheme=https"
    - "traefik.http.middlewares.roosdotclick-insecure-redirect.redirectscheme.permanent=true"
    - "traefik.http.routers.roosdotclick-insecure.middlewares=roosdotclick-insecure-redirect"

    # -- HTTPS entrypoint setup --

    # I want my hostname to be roos.click
    - "traefik.http.routers.roosdotclick.rule=Host(`roos.click`)"
    # On my Traefik container, I have entryPoints.websecure.address=:443 - the name matches up.
    - "traefik.http.routers.roosdotclick.entrypoints=websecure" 
    - "traefik.http.routers.roosdotclick.tls=true"
    # Here's where I specify the resolver
    - "traefik.http.routers.roosdotclick.tls.certresolver=cloudflare-dns-resolver" 
    # I wanted to have a wildcard certificate so I specified these domains for the certificate
    - "traefik.http.routers.roosdotclick.tls.domains[0].main=roos.click"
    - "traefik.http.routers.roosdotclick.tls.domains[0].sans=*.roos.click"
```

Another aspect I really like is the dashboard built in. It doesn't provide much but you can check for errors in your configuration, and you can see all your routes in one central location, and how your routing is going to work. For example for my image endpoint, it shows my custom headers and prefix add that I'm using to redirect to a S3 store (I'm using Minio as I detailed [in my last post]({{< ref "website_build.md" >}})).

{{< figure src="/content/WebsiteBuild_TraefikDash1.png" caption="Dashboard entry showing `i.rooster212.com` router setup" >}}

Traefik for me works a lot better than NGINX, and I will be using it more in the future where I can. It may not have the same performance level but for my uses it's much simpler to setup and debug, and the dashboard is useful. The routing also seems more reliable, I often ran into NGINX gotchas.