---
title: "Running Kali Linux in a Proxmox container"
last_modified_at: 2022-07-15T17:00:00
categories:
  - Blog
  - Video
tags:
  - Kali-Linux
  - Proxmox
videoid: hmwCxD4-WOg
videotitle: Kali Linux in a Proxmox Container
toc: true
header:
  teaser: /assets/images/thumbnails/hmwCxD4-WOg.jpg
---

## KALI LINUX in a PROXMOX CONTAINER!

In this Kali Linux episode we will run Kali Linux with a graphical desktop environment such as XFCE or MATE inside an LXC container on Proxmox. We'll talk about Remote Access using VNC, RDP or X2go. We'll have a look at container templates and how to tweak the network. Last but not least I'll talk about self hosting ideas.
#kalilinux #proxmox

<a href="https://www.youtube.com/watch?v={{page.videoid}}"><img src="/assets/images/thumbnails/{{page.videoid}}.jpg">Watch the video on YouTube</a>

<details>
	<summary>Click to view the entire transcript</summary>
This is the fifth episode of the Kali Linux drop in series on my channel – today we will see how we can run Kali Linux – or any other Linux distribution with a graphical interface – in a container on Proxmox. If you don’t know what Proxmox is then please have a look at one of my earlier videos – link up here – and just once more : In this video I will NOT show you how to “hack xyz with Kali Linux” – If you want to get to grips with ethical hacking then my recommendation is this video by fellow Youtuber Heath Adams. 12 hours of free online course – sliced into chapters. Definitely worth watching. But first – why would anyone want to use Proxmox ? And why would you want to run Kali on it ? And why would we want to run it in a container and not in a VM ? Wow – so many questions – let me answer the last one first – Why container and not VM ? Guys, use the chapters if you want to skip ahead!

A VM – a virtual machine – has something that a container has not: Hardware emulation. That means things like USB ports or a display for instance. And those features consume three things that we are always short of: CPU, memory and/or disk space. A container is much more lightweight. It needs less resources. You can easily run 10 or 20 containers on a server that would be able to run 2 Vms at best.

So why would anyone want to use Proxmox then ? I can’t speak for anyone but myself – I use Proxmox because it gives me a simple way to run Vms and containers with very moderate hardware requirements – an old laptop with a broken display or keyboard will do. I have a small i5 in the basement running loads of containers. And I also have a second machine running a Windows VM on Proxmox that I use to cut videos – remotely, with GPU pass through and remote access with NoMachine. And to answer the last question – why run Kali on it ? Well, as Proxmox is designed to run headless, that means without display and keyboard, I keep it always on – my home automation runs on it and so does Kali. Whenever I need or want it I can just connect to it – pretty much as if it were in the cloud. Just I am running it in house. I am self hosting it. Very convenient. Always on, on demand. 

Maybe a last question here. If we want to use Container technology then why would we not run it with Docker rather than with Proxmox ? Well, that’s mainly related to what I call the „philosophy“ of each product. Docker has been designed to virtualize single processes and to make it easy to replicate them. You would run that one container in Docker and if need would be, then you could spawn 10 or 100 or 1000 of identical containers on the same host. The philosophy of Proxmox with regards to containers is more similar to Virtual machines actually. Even though we run a container, we can do a lot of the nice things which we usually only can do with VMs. Such as snapshots. Or backups. Also I personally find it much easier to manage things like storage and network on Proxmox. Still – the same challenges arise with containers in Proxmox like we had them in WSL2 or in Docker – namely the fact that containers have no display for example – we therefore need a way to access a graphical environment without a display. But if you have watched the previous episodes then you know the solution. We can easily install a graphical environment such as xfce or mate or icewm and then access it remotely using protocols like rdp, vnc or x2go. So let’s do that.

I will not show how to install Proxmox here. This is really covered in my previous video. Quick tip here – I like to use Ventoy if I have to install a machine from scratch – there is a short toolbox video on my channel – link up here.

(Create the container in Proxmox)

Creating a container in Proxmox is a guided process. From the GUI you click on create CT, then you answer all the questions such as the template you want to use – I’ll talk about that one in a minute, then the disk storage and size, the number of processor cores, the amount of memory you want to give that container and which network adapter it should have. This is a huge difference compared to the process of creating a container under WSL2 or Docker. You either don’t have that flexibility there or it’s much harder to achieve. The cool thing is – if you realize that your container needs more juice then you can actually even change those parameters on the fly – while it’s running! Just before I do that – in Proxmox I need a template to start from. You can actually pull a large variety of templates from the Proxmox Internet servers, such as Debian, Ubuntu, Centos and so on – as well as a huge variety of preconfigured turnkey linux containers. And as you do know that Kali is Debian you might as well just start with Debian and then change the installation sources like we did in the first episode. Or – you can pull ready made images from alternate sources and then copy them into the templates directory on any storage that you have on your Proxmox Server. Just a little warning here – the source you pull the template from needs to be a source that you absolutely trust, right ? If this template contains some sort of malware then you have close to no chance to discover them. So I would really recommend staying as close as possible to official sources if you can.

(What are templates?)

On that note – those templates – what are they? Actually they are just a zipped version of the root file system of a container that had been built by someone else. When you install from a template, then this root file system gets unzipped into the file system of the new container. Little spoiler here: That root file system is quite similar to what you have in Docker when you pull a docker image or when you install a WSL2 distribution on Windows – if you want then we might have a look in a follow up episode on how we can migrate Docker containers over to Proxmox or Proxmox containers over to WSL2 or whichever way you want. Leave me a comment here on youtube please if you are interested!

(About Networking)

But let’s focus on that container in Proxmox here. Once I have answered all the wizard’s questions and started the container, then I can access the shell from the Proxmox GUI or of course I can ssh into the container. And here comes the first big advantage – The container has its own IP address. It acts as if it was a physical machine from a network perspective. On my router I can see it as a separate device really. So how do I get a graphical client then ? If I launch a console from the Proxmox Interface, then I only get a shell – very similar to WSL2 – that’s because the container does not have a display of course.

(Installing the software)

Let’s do the same thing like we did in the previous videos – namely install the kali packages and a desktop environment, let’s use xfce so that we get the familiar look. apt install kali-desktop-xfce. Takes a couple of minutes and then we are all set. Now – how will we access this remotely ? We can of course use RDP or VNC like in the last episodes or – this time let’s use another client that I want to show you which comes in really handy in LAN environments. That’s x2go. Let me explain that quickly first.

(about x2go)

X2go does something similar to launching an X11 app over ssh. It connects to the server via ssh and then launches an X2go server session there. That server process holds the virtual display of the X11 session so to say. X2go then uses the NX protocol to mirror that display on my local host. From a network perspective however it would only be using ssh. That’s a big advantage. Of course you could do the same by creating an ssh Tunnel with for example ssh -L 10000:localhost:3398 and then rdp to port 10000 on your localhost. Or with a client like Remmina for Linux or mremoteng for windows you could do that in a row. However – I consider x2go a nice alternative because it’s available for Windows, for the MAC and for Linux and also it spawns a real session, so you could actually have multiple users connect from multiple machines into the same container, right. I might connect as user1 from Linux and someone else as user2 from Windows for example.

(Installing x2goserver)

On the server side, so that’s inside our new Kali container, all we have to do is install the x2goserver by typing apt install x2goserver. On the client side we have to get the client for the Operating system that we are using. Check out their website on how to do that for your OS. Once I create the session, I can then specify which Desktop environment I want to launch. That could be xfce or it could be mate or Icewm. I can in fact install all of them if I wanted and then chose which one to launch. That’s another advantage of x2go, right ? I can chose the desktop environment and I do not have to modify the xrdp.ini file on the server.

(Advantages of using headless servers)

There’s a huge advantage of using headless servers like Proxmox. If I ran Kali on my local laptop let’s say inside wsl2 or with virtualbox, then once I switch off my laptop, Kali will be gone as well. Obviously. If I run it on a Server with Proxmox, then I could have that server sitting in the basement where it can run 24 hours a day and if I had a task that takes longer, such as brute forcing a password or the like, then I would just launch the process, close the client but the process would continue on that remote machine. Pretty much as if I were running it in the cloud, just that the server does not sit in a data center of a provider, but rather in my own home – my own personal cloud really. That’s what self hosting is about really, right? And I can of course protect the containers with a firewall, put them into different network segments and so on, and so on.

(about self hosting)

On that note – self hosting. If you are interested in hosting your own software in virtual environments like Proxmox and you want to get some inspiration on what you may actually use as software – there’s a couple of really good starting points such as the r/selfhosted on reddit or the awesome-selfhosted list on github as well as the awesome-sysadmin list there. Also – maybe checkout fellow youtuber brian from awesome opensource. He’s got some nice videos as well. What am I hosting on my proxmox ? Well there’s the things that need to be running like home automation with FHEM, the satellite TV with vdr, ansible and rundeck for automation, zabbix for monitoring, gitea as a git server, there’s an mdns repeater that allows me to airprint and airplay over network boundaries, plus on the second machine I have an entire sandbox – that’s an isolated test environment with one entry point  - that’s the sandbox client and an exit point – that’s the sandbox router – and a lot of machines in between as well as of course some Windows VMs which I used to make the WSL2 episodes.

That’s it guys – that’s all I wanted to show you today. I hope you liked the episode. If so – likes are much appreciated – anyhow – many thanks for watching. Please do subscribe and leave me a comment and maybe see you next Sunday on the discord server. Stay safe, stay healthy, bye for now.
</details>
