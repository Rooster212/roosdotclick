---
title: "roos.click - the why, the how"
date: 2021-08-08T17:30:00Z
tags: ["tech","coding"]
slug: 'roos-click-why-and-how'
draft: false
---

Having a website can be a bit of a chore. Some people don't bother at all and I get that. It can be a chore to host, maintain, keep secure, deploy etc.

Of course there are options such as Squarespace and Wix, which give you the 'drag and drop' site and could offer things like payments and shop fronts. Then there are blogging sites of which Medium is the one I see used the most. And this probably works for the less tech savvy or people that are just wanting to get content 'out there' in a text format; other formats like video and podcasts are of course also popular.

### Requirements

For me, I had a few 'requirements' in mind - none of them are super strict.

1. **Customisable**. I want to be able to write code if I want to customise something. Preferably in a language I like to work with (that list is probably Javascript, Typescript, Golang, C#...)
1. **Easy to update**. If I'm going to put content out in a text form, I want it to be easy to add new content. I don't want barriers to entry, I don't want to have a manual deployment process. Of course this can be fixed with CI via GitLab or GitHub (which I'm intimately familiar with!) but it's maintenance. Which brings me to...
1. **Maintainable**. If it breaks, how likely am I to fix it when adding content is my primary goal at the weekend or after work? How likely am I going to be to update it with security patches if there are any needed? I had this with my previous site (which was an Express hosted site in a Docker container) - eventually I tried to upgrade some of the dependencies after a couple of years and it was basically impossible.
1. **Static files**. A site that builds into a set of static files is my preferred manner of hosting. It gives a lot of flexibility, and allows for a lot of hosting options. I've got experience with hosting sites via NGINX, via AWS S3 (utilising CloudFront) and even IIS... but that was a long time ago. There are loads of options for this but it gives flexibility.

So after my previous experience of rebuilding [my sisters website](https://gabyroos.com/) using [Hugo](https://gohugo.io/), I decided to tackle two birds with one stone - at the time, I had no automatic deployment in place for that site. I really enjoy working with Hugo and it is highly customisable. I'm not going to go into much detail as [you can review the code on GitLab](https://gitlab.com/Rooster212/roosdotclick).

### Hosting

As mentioned above, there are loads of options for hosting.

1. **AWS S3 and CloudFront**. Get all the advantages of AWS which I'm quite familiar with. Downside is definitely cost - I have an AWS account, and I have used it for some testing and playing with things. The biggest issue is cost. Simply having a hosted zone with a domain in is $0.60 a month. That's before you've even done anything with it!
1. **GitHub/GitLab pages**. I've never used this option but I do know it's quite popular.
1. **CloudFlare pages**. This is a relatively new option, however when I started on this it was in private beta. I requested access as soon as I saw it was available.
1. **Self hosting**. Considering I run a few servers here and there (at home and remotely) mostly running Docker, it'd be easy to put this on a server.

In the end, I decided to give CloudFlare Pages a go. I've used CloudFlare for DNS and to obscure my IP before, so this seemed like an obvious choice.

It was surprisingly easy. They've even integrated CI of some sort, which was a little tricky to get working (which I'll cover in a sec).

### CloudFlare Pages

Not going into a load of detail (as really it's not needed) it goes a bit like this.

#### 1 - Go to the pages console

The Pages console doesn't live strictly under the actual domains. It lives in a seperate part in the console (where you can choose Teams or Registrar etc.)

{{< figure src="/content/website_build/01_cloudflare_pages.png" caption="CloudFlare pages button in console" >}}

#### 2 - Pick your site

My first gripe with CloudFlare pages - it only supports GitHub. A bit annoying - I like GitHub but I prefer GitLab's CI and UI. No problem - I mirror my repo to GitHub and pick it in the console:

{{< figure src="/content/website_build/02_cloudflare_pages_create.png" caption="Pick your site console in CloudFlare dashboard" >}}

### 3 - Sort your build

To put this together I couldn't go back through the steps for my site, however...

The build is great. You connect it to GitHub and it will automatically build versions of your site when you push to the main branch. The 'longest' bit of the process for me was that it was _too_ simple. Long story short - it didn't work initially. After a lot of to and fro, it turns out the default version of Hugo used to build didn't match mine (at the time, it was trying to use a version that was a couple of years old!). It took me a bit of research to work out that I can set the Hugo version using an environment variable.

{{< figure src="/content/website_build/03_cloudflare_pages_hugo_version.png" caption="Environment varible settings" >}}

Then tada... it works!

This isn't meant to be a conclusive overview of CloudFlare pages but it's definitely a nice way of working. It has lots of other options I haven't gone through, and it can do a 'build' per commit which is a nice option if you're iterating on something and are showing it to someone before putting it live. But, check out [CloudFlare Pages](https://pages.cloudflare.com/) in more detail for everything. It really is a nice experience for a static site.