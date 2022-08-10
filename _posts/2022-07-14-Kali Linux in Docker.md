---
title: "Kali Linux with GUI in a Docker container"
last_modified_at: 2022-07-14T17:00:00
categories:
  - Blog
  - Video
tags:
  - Kali-Linux
  - Docker
videoid: JGgu8jVTejk
videotitle: Running Kali Linux in a Docker container
toc: true
header:
  teaser: /assets/images/thumbnails/JGgu8jVTejk.jpg
---

## Kali Linux in a Docker Container - with Graphical Desktop!

We'll run Kali Linux ( or any other Linux distribution) with a graphical user environment such as xfce or KDE or mate inside a Docker Container! I'll explain briefly Docker images, Docker containers and Dockerfiles. I provide you with a Dockerfile on my github server that allows you to chose the X11 desktop environment, the remote access client to use (VNC, RDP, x2go), the Kali software packages to install and which network to use (bridged, host). #kalilinux #docker

<a href="https://www.youtube.com/watch?v={{page.videoid}}"><img src="/assets/images/thumbnails/{{page.videoid}}.jpg">Watch the video on YouTube</a>

<details>
	<summary>Click to view the entire transcript</summary>
Welcome to the fourth Kali episode. Today I want to show you how you can run Kali Linux or basically any other Linux distro with a graphical interface in a Docker container. Spoiler: there is a script and a Dockerfile on my Github for you that will automate this. Use the chapters if you want to skip or fast forward. First question – like usual – why would you want to do that ? Here’s a blue print. Let’s say we have this server here and we want to get the maximum out of it. Whether it is a VPS that you rent in the cloud or an old laptop or a juicy machine sitting in a data center -  It doesn’t matter. The point is – if you want to run multiple things on one machine – then sooner or later you’ll come across virtualization technologies like VMs or containers. If you’re using Docker then you might for example have let’s say a web server running in a container. You’d have not only one but maybe 10 or 1000 of those. Proven technology, works incredibly well. All you’d have to do with docker is to add another container and run Kali inside of it. We’ll see how to do that. It’s really not difficult. 

Before I get a lot of comments saying “Marc, Docker has not been designed to run a full Linux distribution” - Yes - Docker has not been made for that. But it has been designed so well that you can use it this way. Still – Docker is not a virtual machine platform. It had been designed for process virtualization and scalability. If you want to run a full Distro in a container on a separate machine and access it remotely then I strongly recommend using Proxmox for that. It’s free, it can do VMs and containers, it can run on any not too old 64 bit hardware and it is really easy to set up and use. We’ll do that in the next episode actually. You might want to subscribe in order to not miss out on that one ;-) But let me briefly talk about Docker here. Again – use the chapters. 

And once more – this video is not about “hacking xyz with Kali Linux” – don’t fall for the click bait here guys. Offensive Security or short OffSec (who are the creators of Kali Linux) have courses and certifications. The link to their YouTube channel is up here. They do have a discord server as well.  They are not paying me money or anything for me to say that by the way. No affiliation whatsoever. Just to clarify.

(about Docker)

In Docker we have two components that are difficult to grasp when you start doing Docker. One is the docker image and the other one is the Docker container. What are they ? Think about it this way: When you go to a computer shop or to a website and buy or download a software, then you would presumably  get either a CD, DVD or a File like setup.exe etc. That is the real full software, but you can’t really do anything with it – you need to install it first in order to use it. In other words: the Setup file or DVD is stateless – it’s not reading or writing any data and doesn’t change between today and tomorrow. Once you install the software, then that installation becomes stateful. That means, you can use it, it’s reading and writing data. In Docker terms, the installation CD would be the image and the container would be the installed software which is up and running. 

A docker container typically is built by a Docker file. That’s basically just a written recipe how to build the container. Usually you start from a generic image such as debian:latest or kali-rolling in our example. That’s the FROM soandso statement. And then you say – on top of that image, run the following commands – install software, create a user and so on. At the end you typically tell it which command to run once it’s done building it.

Now – the Docker “philosophy” is such that if the container does anything weird or is not working as expected, then you would just scrap it and launch a new one. So in that perspective it’s really very different from let’s say a virtual machine which you would maybe reboot but rather keep in its current state. As with Docker, you have the docker file, you basically have a recipe to build another one that is identical to the one that you scrapped. With one difference: All the data that you have added to it is gone. We’ll see how we can address that in a minute.

(Getting Docker)

Cool – so we need a Linux host – that can be a virtual machine, a physical machine, could be a Raspberry Pi, it can be a virtual private server VPS that we rent somewhere in the cloud or it can even be a container running under Proxmox for example. Anyway – we need a Linux machine running docker. I personally prefer Debian Linux for this for the following reason: under Debian you can install the docker.io package. That is maintained by Debian and installs all the dependencies in separate packages. On other distributions you would have to install the docker-ce package. CE stands for community edition. While docker-ce is the docker certified version, I find it a bit more difficult to install and maintain because you would need the docker repositories. But hey – it’s up to you to decide. Leave me a comment if you have any preferences. So all I do on Debian really is sudo apt install docker.io. As a GUI for Docker I still love portainer which in turn runs inside docker. In order to get it up and running I just need to run that one-liner here which you find in the description of course. I won’t go too deep on portainer guys, there’s many videos on that – it will just help us lookup or change things later if we wanted to.

(Installing Kali in Docker)

Awesome – now we have Docker. How do we get Kali to run inside of it? And how do we tell Docker which tools to install into it ? I have actually created a simple Dockerfile for you guys which you can find on my github repository. The link is – like always – in the description of the video. Furthermore I have written a little installation script that allows you to chose the desktop environment that you want to use, be it mate or xfce or whatever. You may also chose the remote access software which you want to use, be it xrdp or vnc or x2go. Plus you can chose which packages you want to install – like core, default, everything, headless and so on. Instructions on how to use it are on the github.

(Accessing Kali)

Once the installation is finished, you can connect to that container’s IP address using your favorite remote access software – Windows Terminal Services Client, Remmina, TigerVNC, x2go – whatever you like. The user name for all operations is kaliuser and all passwords are set to onemarcfifty – all in minor case. Here we go – we have a nice graphical Kali session running inside Docker. Just a few remarks here – I have not tested KDE and Gnome. Gnome will presumably not work as it has dependencies on systemd. Also there seem to be a couple of drawing issues with xfce and x2go which I haven’t figured out yet probably due to the bells and whistles it has in the menu bar – like the System monitor and so on – the most reliable GUI with x2go at the moment seems to be with the mate desktop. XFCE is however running totally fine and very fast actually with VNC.

(Networking)

Let’s talk about the network quickly. I have not specified any privileges or rather capabilities for the docker container. That means that some things presumably don’t work, such as Wireshark. You would need to add the corresponding capabilities to the docker create statement in the build script. I might add that as a menu option in the future. But everything that’s just outgoing works – even nmap is not a problem.

By default, a docker container uses the bridge network. That’s actually a NATed network behind the host’s network. That means that if you want to access the container from the outside world, then you would have to map that port to the host’s network.  If you want to run stuff that needs direct network access inside the container such as arpspoof or wireshark, then you would however need direct network access. That’s what the host network does. There would be more choices. You could actually assign a macvlan network to the container and have it behave like a real physical host. But guys – I will not go into details on this as I have already made two videos about Docker networking which you can find up here or in the description again. For arpspoof and the like you should be fine with option 2. If you chose option1 – that means the bridge network - and run it on a remote host, then I am just mapping the default ports like they are defined in the script to the host. 
If you want to run this on a server in the cloud – so on a VPS – then you should however really use the option 1 which is the bridge network and have your cloud server secured with a Firewall and VPN or at least with an ssh tunnel. If you want me to make a video on that – please drop me a comment. 

(Volumes)

I told you that if you would scrap the container and build it from scratch again, then all the data inside the old container would be gone. And that’s “gone” like in “gone forever” unfortunately. We can however make storage persistent in Docker. That means we kind of map a portion of the Docker host storage to the container. That can be done using so called bind mounts or using Docker Volumes. A bind mount would just map an existing directory or file to a directory or file inside the container. The script actually creates a Docker Volume called kaliuser_data and maps the kaliuser’s home directory inside the container to it. When you recreate the container, then it would have all the config files there at startup. Here’s what that looks like in the portainer GUI. You can see that kaliuser volume there. On the docker host by default this would reside under /var/lib/docker/volumes by default. However, everything outside that home directory such as software packages that you installed and so on would be gone.


Awesome. That’s it for today guys. I hope you liked that episode. If so – thumbs up is appreciated ! If not, leave me a comment what you would have liked to see and in either case – many thanks for watching. Don’t forget to subscribe so that you don’t miss out on the next episodes. Stay safe, stay healthy, bye for now !
</details>
