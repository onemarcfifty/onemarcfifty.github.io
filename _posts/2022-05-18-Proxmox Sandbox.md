---
title: "A virtual network Sandbox in Proxmox"
last_modified_at: 2022-05-18T17:00:00
categories:
  - Blog
tags:
  - Proxmox
toc: true
header:
    teaser: /assets/images/Channelart11b.png
---

(A video version of this blog is available on Youtube : [Watch it on YouTube](https://youtu.be/R67wEo2V710) 

## Why would I want a Sandbox?

Sometimes I need to have a test sandbox. A virtual representation of my LAN really. Why? Here are some things that I want to rather do in a sandbox than in my real LAN:

- Test a VPN connection from the Internet
- Test port-forwarding from the Internet
- Install and test software (e.g. for my videos)
- play around and break (and possibly fix) things
- Test new stuff like IPv6, DNS settings, DHCP, VLANs etc.

## Why Proxmox and not Virtualbox?

In October 2020 I had published a video where we did this in in Virtualbox: [Watch it on YouTube](https://youtu.be/JwPD1GWw5go) At that time we used a Virtual Machine (VM) to access a virtual network running on our local machine.

Unfortunately and unlike the  [Proxmox Virtual Environment](https://en.wikipedia.org/wiki/Proxmox_Virtual_Environment), Virtualbox can't do containers natively. VMs use a lot of resources. Containers are much more lightweight.

The blueprint of a container-based solution in Proxmox is as follows:

- A **virtual network** that only exists on the Proxmox Server itself and is **invisible from the outside**
- A **container** running a **graphical** Desktop Environment (MATE) under Linux
- A **perimeter router** connecting the Virtual and the real LAN running OpenWrt (you may use anything else if you want such as pfSense or OPNSense or Mikrotik RouterOS)

## OK, convinced. How to build this?

Here are the steps to build this: (You need to have a Proxmox Server with Internet access ready, also please download and install the [X2GO Software](https://wiki.x2go.org/doku.php/download:start) on your PC or Laptop - we will use this as a client software) Please note that if you run this on the Mac then you will also need the XQuartz software.

### Build the client container

#### creating the Container

First we will start with the Sandbox Client Container running a graphical environment under Linux. 

- Create a Container ("Create CT" button in Proxmox)
- Use Debian 11 as a template
- Give it 1 GB of RAM
- Network=your LAN
- IPV4=DHCP (I assume you use DHCP in your network)


Don't start the container yet. Go to the options of the container and tick the following boxes:

- Nesting
- FUSE

#### installing the software in the container

Now start the container, then open a console in the container and execute:

	apt update
	apt install mate wget curl sudo x2goserver firefox-esr
	systemctl start x2goserver
	systemctl enable x2goserver
	
This will install the graphical environment MATE, Firefox as a web browser, x2goserver for graphical access from your PC and a couple of tools such as sudo and wget. The last two statements start and enable the X2GO Server at boot time.

#### adding a non-root user

Next we need to create a non-root user on the Container. We should really not run graphical environments and web browsers as root:

	useradd -m -G sudo -s /bin/bash marc
	passwd marc

Of course you don't have to call your user "marc" - change this to anything you want. The options of this command create a home directory (-m), add the user to the sudo group (-G sudo) and define bash as the shell for the user (-s /bin/bash). The passwd command prompts you for a password for the user.

You can try out if everything works correctly by exiting the shell and loging in as the new user

	# leave the root shell
	exit
	# now log in with the new user and pass and see if you can sudo
	sudo bash
	# you should now get a shell with root access

#### connecting to the container

Now you can already set up your X2GO client. Go to File-New Session in the X2GO client software and create a new session. Give it a fancy name, set the user name / login to the username which we have just created. Specify the hostname or its IP address on the connection. If you are unsure of the IP address of your container you may check that in the Proxmox console with

	ip -br addr
	
 Save the new session and connect to it. You will be asked for the password and should then be able to log in to the MATE environment of the sandbox. 

### Build the virtual network

Building a virtual network in Proxmox is just a handful of clicks really:

- Browse to System-Network on your Proxmox VE
- Click on Create-Linux Bridge
- Give it a name, e.g. "vmbr9999"
- optional: Tick the box "VLAN aware"
- confirm with the "Create" button
- click on the "Apply Configuration" button

If you get an error saying you need ifupdown2 then log into a root shell on the Proxmox PVE and execute

	apt install ifupdown2

That's it - you now have a virtual network inside Proxmox called "vmbr9999"

### Build the perimeter router


#### build the VM

Now click on "Create VM" in the Proxmox VE interface. 

- General tab: VM ID: any ID (I used 999) Name: any DNS compatible name (I used perimeter-owrt)
- OS Tab: Do not use any media
- System: default values
- Hard disk: default values (we will throw the HD away and use the Image which we will import in the next step)
- CPU: 1 Socket, 1 Core
- Memory: 512 MB
- Network: use the bridge that we have defined above (e.g. vmbr9999)

Do not start the VM yet, we first need to downoad and import an OpenWrt image 

#### Download and import the OpenWrt image

Please open a shell on your ProxMox server and download the OpenWrt image: 

	wget https://downloads.openwrt.org/releases/21.02.3/targets/x86/64/openwrt-21.02.3-x86-64-generic-ext4-combined.img.gz
	gunzip openwrt-21.02.3-x86-64-generic-ext4-combined.img.gz
	# now import the disk. Replace "999" with the ID of your VM. You may replace "local-lvm" with another storage
	qm importdisk 999 openwrt-21.02.3-x86-64-generic-ext4-combined.img local-lvm

#### attach the disk

Click on the VM (e.g. "999 (perimeter-owrt)" ) in your Proxmox VE console and click on the "Hardware" option.

You can see an unused disk 0 - that is the newly imported disk image. Before we attach it, we need to detach the disk which we created during VM creation. There should be a hard disk (resumably 32GB, SCSI) in the list. Select it and click "detach". Once it appears as "unused Disk 1" in the list, you can delete it.

Now attach the unused disk0. double-click on it. Select IDE 0 as the controller and save.

Last but not least, browse to the "options" item of the VM, then double click on "Boot Order", unselect all entries, move the ide0 to the top of the list and tick the  "enabled" box.

One more thing - you need to give the VM a second Network adapter (in fact the virtual "WAN" which in turn is our physical LAN). Go to the hardware options of the VM, click "Add", then "Network device" and select your hysical LAN adapter as a second adapter.

You can now start the router image and should see it boot in the shell window.

### connect everything

At this point we have all machines and the network up and running. Let's connect everything. First you need to add a second network adapter to the sandbox client container. Select it in Proxmox, click on "Network" , then "Add".

- Name: eth1 (I assume your first adapter is called "eth0")
- IPV4: DHCP
- Bridge: vmbr999 (or whatever you have called your virtual bridge)

Now connect to the sandbox client using the X2GO software and open a terminal inside the MATE environment (Applications-System Tools-MATE Terminal). Inside the terminal window do the following to restart the network

	sudo systemctl restart networking
	ip -br addr

If that fails, reboot the container. You should now have two IP addresses. One from your physical LAN and one from the virtual 192.168.1.x LAN

Note: If your physical LAN happens to be in the 192.168.1.x range then you would need to move the virtual LAN to another range, e.g. the 192.168.100.x range. In order to do this, open a shell on the OpenWrt Virtual image (999 - perimeter-owrt) and type the following commands:

	
	uci set network.lan.ipaddr='192.168.100.1'
	uci commit
	reboot

 You should now be able to open a web browser inside the sandbox client and browse to the 192.168.1.1 address of your virtual router and open the OpenWrt LuCI page.
 
 there is one last change that we need to do. At the moment we have two network interfaces on the sandbox client (the MATE linux). But we haven't explicitely told it which route to take. We want the sandbox client to route exclusively through the sandbox. Inside the MATE terminal type the following to verify:
 
	 ip route
 
 Presumably your default route goes through the physical LAN. In order to change this, we can modify the DHCP client config. Inside the MATE terminal type
 
	 sudo pluma /etc/dhcp/dhclient.conf
 
 That will open MATE's text editor (Pluma) as root. Remove everything from that file and replace it with the following content:
 
 
	# Configuration file for /sbin/dhclient.
	#
	
	option rfc3442-classless-static-routes code 121 = array of unsigned integer 8;
	
	send host-name = gethostname();
	
	# The LAN interface is used ONLY to view the X2GO server. We do NOT
	# need a default gateway and the like here
	
	interface "eth0" {
	        request subnet-mask, broadcast-address, time-offset, 
	        host-name,
	        interface-mtu,
	        ntp-servers;
	}
	
	# our virtual bridge in turn, i.e. the “Sandbox” interface is the virtual LAN
	# and hence the network we should route through
	
	interface "eth1" {
	        request subnet-mask, broadcast-address, time-offset, routers,
	        domain-name, domain-name-servers, domain-search, host-name,
	        dhcp6.name-servers, dhcp6.domain-search, dhcp6.fqdn, dhcp6.sntp-servers,
	        netbios-name-servers, netbios-scope, interface-mtu,
	        rfc3442-classless-static-routes, ntp-servers;
	}
	
This will prevent the sandbox client from requesting a route from the physical LAN and have a route exclusively through the Sandbox. It will also force the client to use the sandbox router as DNS server. Save and Close Pluma. Again, in MATE terminal type

	sudo systemctl restart networking
	ip route

and check that the route now goes through the sandbox.

## Why x2GO and not VNC/RDP?

The nice thing with X2GO is that it uses ssh. In fact it tunnels X11 over ssh. Plus you can easily selet the graphical environment to use (KDE, Gnome, ICEWM, MATE etc.). Also, it allows you to use the clipboard and - how should I put it - it feels "real", i.e. no remarkable latency - at least in the LAN if you don't do video or the like. Also there are clients available for Windows, Linux and MacOS. I like it ;-)

