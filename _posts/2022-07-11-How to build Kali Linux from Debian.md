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
