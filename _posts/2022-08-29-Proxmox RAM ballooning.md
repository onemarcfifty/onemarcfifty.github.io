---
title: "How to dynamically allocate RAM to a VM on Proxmox"
last_modified_at: 2022-08-29T17:00:00
categories:
  - Blog
  - Video
tags:
  - Proxmox
videoid: oqd-j3XgvJk
videotitle: "How to dynamically allocate RAM to a VM on Proxmox #shorts #proxmox #virtualmachines"
toc: true
header:
  teaser: /assets/images/thumbnails/oqd-j3XgvJk.jpg
---

## How to dynamically allocate RAM to a VM on Proxmox 

#shorts #proxmox #virtualmachines

The ballooning option allows Proxmox VE (https://www.proxmox.com) to manage the RAM of a Virtual Machine dynamically. Rather than having a fixed amount of RAM for each VM on Proxmox, you can use the dynamic RAM allocation with a so called ballooning device. That means that you can run more VMs on one single Proxmox Server.

<a href="https://www.youtube.com/watch?v={{page.videoid}}"><img src="/assets/images/thumbnails/{{page.videoid}}.jpg">Watch the video on YouTube</a>

<details>
	<summary>Click to view the entire transcript</summary>
If I create a virtual machine on Proxmox then I can give it a certain amount of RAM. But do I really need to block the whole RAM for the machine even if it doesn’t use it ? No ­you don’t – just tick the box “Ballooning device” on the Memory options of the VM. This way Proxmox allocates memory dynamically. There’s a minimum value that always gets allocated and a maximum. This way you can run more VMs on one Proxmox machine if you’re not using all of them at the same time. Thanks for watching. Please Like and subscribe. 
</details>
