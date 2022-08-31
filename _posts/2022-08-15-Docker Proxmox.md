---
title: "Run Docker in a Proxmox LXC Container"
last_modified_at: 2022-08-15T17:00:00
categories:
  - Blog
  - Video
tags:
  - Docker
  - Proxmox
videoid: uvecfr4na-8
videotitle: "Run Docker in a Proxmox LXC Container #shorts #docker #proxmox"
toc: true
header:
  teaser: /assets/images/thumbnails/uvecfr4na-8.jpg
---

## Run Docker in a Proxmox LXC Container 

#shorts #docker #proxmox

If you want to run Docker on Proxmox VE (https://www.proxmox.com) then the documentation suggests you run Docker inside a VM. But if you tick the right two or three boxes then you can easily run Docker inside an LXC Container on Proxmox VE. The keys to select are nesting and keyctl

<a href="https://www.youtube.com/watch?v={{page.videoid}}"><img src="/assets/images/thumbnails/{{page.videoid}}.jpg">Watch the video on YouTube</a>

<details>
	<summary>Click to view the entire transcript</summary>
So if you want to run Docker on Proxmox, then the recommended way to do that is to create a virtual machine and then run Docker inside that VM – but – really ? Run a Linux VM on Linux and then run Docker inside that VM? There must be a better solution. Yes – Even though t­he documentation doesn’t say so, you can run Docker inside a container on Proxmox. All you have to do is that either you make the container privileged by unticking “Unprivileged Container” or – if you want an unprivileged container – you go to the container’s options and tick those two boxes under Features: keyctl and Nesting. Now you’re ready to go. If you installed Debian 11 for example, then go to the shell and install docker by typing apt install docker.io. Done. Thanks for watching. Please Like and subscribe. 
</details>
