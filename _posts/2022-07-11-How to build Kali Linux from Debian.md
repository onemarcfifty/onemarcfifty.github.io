---
title: "How to build Kali Linux from Debian"
last_modified_at: 2022-07-11T17:00:00
categories:
  - Blog
  - Video
tags:
  - Kali-Linux
videoid: 9TNEpaqDfwg
videotitle: DIY MESH WiFi with batman-adv and OpenWrt
toc: true
header:
  teaser: /assets/images/thumbnails/9TNEpaqDfwg.jpg
---

## How to build Kali Linux from Debian

You know what Kali Linux does or what it's good for - but what is it ? Kali is a Linux Distro based on Debian. In this video I show how to build Kali from Debian. We will change the apt installation sources from Debian to Kali and hence tell the Linux distribution to pull everything from the Kali apt repositories. We will then install all the apt packages that make Kali Kali, i.e. kali-defaults, the kali-desktop package etc. On a side note, I'll show how to run X11 apps over ssh plus quickly talk about WSL2 on Windows 11. We will chose an alternate desktop environment (i.e. rather than using xfce4 we will use mate) #kalilinux #x11 #linux #debian

<a href="https://www.youtube.com/watch?v={{page.videoid}}"><img src="/assets/images/thumbnails/{{page.videoid}}.jpg">Watch the video on YouTube</a>

## changing the apt sources

In order to turn a Debian Linux into Kali, we need to change the content of the `/etc/apt/sources.list`

Here's what it should contain:

    deb http://kali.download/kali kali-rolling main non-free contrib

you may do this by typing

    echo 'deb http://kali.download/kali kali-rolling main non-free contrib' > /etc/apt/sources.list


## obtaining the Kali apt key

Now we need a key in order to verify the authnticity of the Kali Linux apt sources:

### Method 1: wget the asc key directly

This is the easiest method if you have wget installed. Just type

    wget https: //archive.kali.org/archive-key.asc -O /etc/apt/trusted.gpg.d/kali-archive-keyring.asc

This will put the key in the right directory

### Method 2: apt-key (deprecated)

You can download the key with apt-key as follows:

    apt update && apt install -y wget curl gnupg2
    apt-key adv --keyserver hkp: //keyserver.ubuntu.com:80 --recv-keys ED444FF07D8D0BF6

### Method 3: Install the Kali Key package

Download the Kali Archive Keyring package from [kali.org](https://http.kali.org/pool/main/k/kali-archive-keyring/kali-archive-keyring_2022.1_all.deb) and run `dpkg -i kali-archive-keyring_2022.1_all.deb`

## Updating and installing the Kali-Linux packages

Now you can update the apt packages source and then turn Kali into Debian by typing:

    sudo apt update
    sudo apt upgrade
    sudo apt install kali-defaults

Choose which Toolset and Desktop environment you want by searching for the right package with

    apt-cache search kali-tools
    apt-cache search kali-linux
    apt-cache search kali-desktop

If you wanted xrdp as a remote access software and the complete Kali Toolset in XFCE4 you would type:

    sudo apt install kali-linux-everything kali-desktop-xfce xrdp

## adding a non-root user

Do not log in as root into the X11 session! In order to create a non-root user that has sudo capabilities type the following in a shell:


    useradd -m -G sudo -s /bin/bash kaliuser
    passwd kaliuser

This adds the user kaliuser, adds it to the sudo group (-G), creates a home directory (-m) and defines a shell (-s). It then lets you define a new password for the user.

<details>
	<summary>Click to view the entire transcript</summary>
I am sure that you know what Kali linux does or what it is good for – it’s a Linux distribution that focuses on security. In other words, it has all the nice hacking tools included. But what is it? I mean, what makes a Kali Linux Kali and not Ubuntu or Mint or CentOS? And why would you care? Let me answer the second question first - Let’s say you wanted to run Kali Linux in the cloud – on a virtual private server (VPS) that you rent in the internet – you pay a dollar or five or ten per month and you have that nice machine sitting somewhere in a data center, maybe in a different country and you want to run Kali on it. But – the provider doesn’t offer Kali as a distribution. And they don’t allow you to upload an iso image or maybe you would need to pay for that. In this scenario it would come in handy to know how to build Kali from scratch. And actually that only takes two minutes really. Because… Kali is Debian. Or based on Debian I should say. And most providers definitely offer you to install Debian. But what is the difference between a vanilla Debian and Kali? Well, there is actually only one small but decisive difference: the installation sources. Let me explain that quickly.

“Debianish” or “GNUish” Linuxes like Debian, Ubuntu, mint, Raspbian or Kali use the advanced package tool (apt) in order to install software. If you want to install a software on those Linuxes then all you have to do is open a terminal or ssh session and type sudo apt install and then the name of the software package that you want to install. Apt will then pull all required packages from the internet and install them on your machine. But how does apt know where to pull the packages from? That information is stored in a simple text file. That file is located in /etc/apt/sources.list. Have a look. On a Debian it looks like this – all sources point to the Debian servers of course. On Ubuntu they point to Ubuntu. And on Kali they point to Kali. So all we have to do in order to make a Debian Linux become a Kali Linux is to change that file so that the sources point to Kali instead of Debian. Let’s do that.

But just before we do that guys – this video is not about “hacking xyz with Kali Linux” – and many of those videos are click bait anyhow. Just because I run Kali Linux that does not make me a hacker. As much as owning a great pen does not turn me into a good painter. If you do however want to get a serious introduction into ethical hacking then I can recommend this video by fellow Youtuber Heath Adams. It’s a free 12 hours long course – but he has added chapters to the video so you can easily pick the 10 or 20 minutes that you’re interested in. Strong recommendation. Also – Offensive Security or short OffSec (who are the creators of Kali Linux) have a youtube channel – The link is up here as well – You might consider taking one of their courses or certifications. They do have a discord server as well. All links are in the description. Back to Kali and Debian.

I have a machine here that is running Debian. It doesn’t matter where it is for this episode – we’ll look into options in the following episodes. Let me turn it into a Kali Linux quickly. If I ran apt update then it would pull all the packages from the Debian sources. Before I can use the Kali sources I just need to have the right key in order to verify their authenticity. That key is public and I can download it from kali.org using the wget command. If wget is not available then try curl or install it with apt install wget. There are various ways to install that key. Check the description of the video. There are at least three which you can try if ever one is not working. Once I have the key then I can just hard overwrite the apt sources.list file. That’s what we now have in the file. And the key has been put here. Let’s run apt update again. As you can see everything is now pulled from the Kali sources. If I type apt upgrade then the Debian packages will be replaced with the Kali packages. Just a quick warning. Do not mix too many sources for apt. try to stick to the default of the distribution. Otherwise you may end up with a so called “Frankendebian”. Google it for more information.
 
Let’s stop here for a second. We know a lot of things already. We know that Kali’s foundation is Debian. We know that they both use apt as the package management system. We know that we need a key to install packages from a repository and we know how to obtain that key and how to change the apt sources file so that it points to the Kali sources. Basically we now know how to build Kali from Debian.

But if we look at the terminal, then we don’t really see a difference, do we? That’s because there are other components that make a Kali Linux look and feel like Kali. The first ones are the Kali meta packages and the second one is the graphical desktop. Let’s take them one by one. There are a lot of packages that you can install which are specific to Kali. If I run apt-cache search kali then I will get a huge list of packages that contain “kali” in the name or in the description. There is actually a page on the kali.org web site that lists and explains the most important ones. But let’s go step by step.

The first package that I want to install is called kali-defaults. It’s a small package. You’ll see what it does in a second. So let’s do apt install kali-defaults. We still don’t see any difference, so let’s quit the shell and log back in. There we go. That’s the Kali look and feel of the shell. We’re getting closer. Actually in 2020 Kali changed from bash to zsh as the default shell. Now for the graphical environment.

When you search on YouTube for Kali Linux then you will find a lot of videos and nearly all of them tell you to do the following: Install xfce4 and then install xrdp. No one really explains why to do it that way. Let me explain. Guys, use the chapters if you want to skip forward. I just want to show you something that you might not know if you are a Windows user.

What we see so far is the shell running in a terminal session. If we want to run graphical applications on Linux then we use the X11 subsystem. This does come in several flavours such as xorg or wayland, but more on this in future episodes. I can then launch a graphical application on my X11 client (which in this case is the Kali machine) and display it on the X11 server (which is the machine which I am sitting in front of). Notice the inversed roles of server and client here. Let’s give it a go and install some basic x11 tools. I run apt install dbus-x11 xauth x11-apps. Log out and log back in with -X and -Y. If I now try to run a very simple x11 application like xeyes then Linux tells me there is no display. That is because Windows does _not_ have an X11 server built in. I would need additional software like Xming to provide that functionality. If I do the same from a Linux workstation then the xeyes application just starts. It’s running on the Kali host and it’s just displayed on my local host. On a Mac you could do this as well if you have Xquartz installed. Ssh can forward the X11 instruction set from one machine to another. That’s what the -X argument of ssh was for. So the way we solve this for Windows is that we run a kind of proxy on the X11 client. Again - that’s the Kali host. That proxy allows us to connect from the outside world using a remote desktop protocol such as RDP which is used by windows. But it doesn’t have to be RDP. It could be VNC as well or NoVNC or X11 over ssh by using a client such as x2go. Or kex. Whichever it is – it will be running the graphical application on the host and kind of mirror or stream the content to our workstation over a network connection in fact. On the other end we need the corresponding client for that protocol. On Windows 11 you can actually run this more or less “natively” – watch what happens if I start the xeyes application in WSL2 – it takes a while and then it comes up. Only difference – the eyes only react to cursor movements inside the window. The Window Manager obviously only controls that window but not the whole desktop. We’ll talk about this more in depth in the third episode.

Now – xeyes is kind of fun but we want to run a full desktop environment. That needs a couple more components. We already have the display server which is X11. The next component is the display manager which handles the graphical login and user session. On Linux they are called gdm or lightdm for example. Plus we want a window manager which actually provides the borders, decorations and minimize, maximize, close buttons etc. And last but not least we want a set of applications, menus, tools and so on. The good news is that we don’t need to worry too much about these components as Kali provides a number of so called meta packages which contain all the required software – a meta package is a package that points to many other packages.

Let’s start with the desktop environment. If I type apt-cache search kali-desktop then I get that list of potential candidates for the job. Unlike in Windows, on Linux I am free to chose here. Feel free to install xfce here, but my personal favorite is still Mate. It has less bells and whistles but I just like the tools it comes with. Try them out – use whichever you feel most comfortable with. The packages do vary in size. You may see this if you type let’s say apt install kali-desktop-xfce – that shows me the disk space which it would allocate after installation. If I interrupt this with no and use Mate or KDE instead then I can see the difference in size. I want mate so let’s go. This will need to install roughly 800 packages in my case. Quick tip here – if your VM has  really tiny disk space then you may as well use icewm – that’s probably the smallest X11 desktop that you can get. Not beautiful in my opinion -  but it does the trick.

Cool – the installation is finished. Now let’s chose a remote access option. For starters lets go with rdp. I’ll do – like all the others on YouTube – apt install xrdp. We’ll have a look into other options in the following episodes. Let me check with ps if it’s running. No it’s not, so I’ll start it using systemctl. On WSL2 systemctl would not work so you would want to do “service xrdp start” instead. Check again. Here it is. Quickly list the IP address with ip -br addr. I should now be able to connect. Here we go. But wait. Which user account should we use ? Please don’t use the root account here. You should never run a complete X11 session as root. Everything you run inside that session will be ran as root as well. If you browse to a website from this session as root and you catch a Trojan as root then you will be owned in less than a second without noticing it. So let’s quickly go back to our terminal and create a user without root privileges. The command to add a user is useradd. Also I want to make sure that I have sudo installed actually. Plus there’s some more parameters such as I want it to be a member of the sudo group and define a shell etc. Guys, the commands and the explanation of the options is in the description of the video. Next I launch the passwd utility in order to specify a password for the user. That’s it. Let’s connect again as the new kaliuser – and – here we go – beautiful Mate session. 

You may have noticed the certificate warning when I connected to the kali box. That comes up because xrdp doesn’t have a certificate or is using a self-signed one. If you wanted to use certificates here then you would have to edit the /etc/xrdp/xrdp.ini file and point it to the right certificate such as a letsencrypt certificate or the like. Ignoring this warning in a local network should not be a problem. But if you connect to a remote machine in the internet then I strongly advise you to use proper certificates. Check the xrdp.ini man pages for more details.

On that note – I would not recommend connecting to a server in the internet over RDP directly. Even though RDP should be encrypted, it’s lacking a strong authentication mechanism. Also – when we see the RDP login screen, then we are already on an application layer. That layer may have a lot of vulnerabilities. It’s better to use authentication on the transport layer with a VPN or at least with an ssh tunnel.

Actually, fellow Youtuber NetworkChuck is doing this in a video where he created a free EC2 instance inside Amazon AWS. The link to the video is up here. Check it out, his videos are fun to watch. However, personally I am a bit reluctant to using AWS instances because I feel that I don’t have full control over if and how much they would charge me. In the given use case with a free EC2 instance at the time of making this video in June 2022 you would get the virtual machine in the so called t2.micro layer for free for one year. Plus you get EBS storage – that’s what it’s called at AWS – up to 30 GB. Going over 30 GB in disk size for that VM or using it longer than 12 months will result in additional cost. By the way, the machine you get has one vCPU and 1 GB of RAM. So it’s not too bad actually. Just watch your credit card bill. Networkchuck actually created a virtual credit card for that. Very innovative. But I personally prefer a fixed price for a fixed service or – even better – an hourly price that is capped to a monthly maximum. Let me know if you want me to make a video on secure remote access to the cloud. Leave me a comment on Youtube!

Good – now the last thing that we need are the kali tools. You may of course install them one by one if you would for example only be interested in metasploit and theharvester for example. But here again we can use the meta packages. If we search the apt cache for kali-tools then we can find a couple of theme based toolsets. And looking for kali-linux shows some preconfigured packages with different sizes of tools pre-bundled. I might go for kali-linux-everything but this will take very long. You can see the result here in my Kali sandbox image. All the tools are here.

Before we close – am I using Kali Linux at all ? Yes, sometimes – when I want to try something out and I know the tool is in Kali – I have a Kali container running in my Proxmox sandbox. But the tools I use on a regular basis – I prefer to install them individually on my desktop Linux – which is Debian as well. Still – it’s a quite easy way to get your hands on some of the tools if you want to try them out. So there is nothing wrong in using it.

In the next episode we will see how we can run Kali inside a VM – that’s more of a “classic” use case. The following episodes will show you other options, such as WSL2, LXC containers and even Docker.

Awesome – that concludes today’s episode. Many thanks for watching. If you liked the video, then please do give it a like on YouTube. Leave me a comment with your use cases and suggestions and maybe see you next Sunday on Discord ? Stay safe, stay healthy, bye for now.

</details>
