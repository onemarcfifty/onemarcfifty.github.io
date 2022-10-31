---
title: "Upgrade and Revert using Grub and efibootmgr"
last_modified_at: 2022-10-31T17:00:00
categories:
  - Blog
  - Video
tags:
  - Networking
  - OpenWrt
videoid: 3UMx7P5n91Y
videotitle: "How I upgraded and reverted Proxmox from 6 to 7 to 6 with UEFI boot and GRUB"
toc: true
header:
  teaser: /assets/images/thumbnails/3UMx7P5n91Y.jpg
---

## Proxmox Upgrade

I wanted to upgrade my [Proxmox VE](https://www.proxmox.com) Server from version 6 to version 7. But I did not want to do this without a **Plan B**, a ***fail back plan***. For this I used GRUB and efibootmgr. I converted a small swap partition into a bootable Linux partition and pivoted the Version 6 to Version 7. When I noticed that things did not work as expected, I was able to revert to version 6 in less than a minute using GRUB, UEFI Bios and efibootmgr.

## Situation before the Upgrade

I have Proxmox VE (Version 6.x) running on my Server. My Proxmox Server has two hard disks, but for this exercise we will only look at the first one. In fact, as the server boots with EFI, there are three partitions on that disk:

	Sector size (logical/physical): 512 bytes / 512 bytes
	I/O size (minimum/optimal): 512 bytes / 512 bytes
	Disklabel type: gpt
	Disk identifier: xxx-xxx-xxx-xxx-xxx

	Device       Start       End   Sectors   Size Type
	/dev/sdb1       34      2047      2014  1007K BIOS boot
	/dev/sdb2     2048   1050623   1048576   512M EFI System
	/dev/sdb3  1050624 937703054 936652431 446.6G Linux LVM

The second partition is the EFI boot partition, and the third partition contains all the LVM volumes. In order to see the pools and volumes, I type `lvs`

	LV                       VG  Attr       LSize   Pool 
	data                     pve twi-aotz-- 340.00g
	root                     pve -wi-a-----  82.00g
	swap                     pve -wi-a-----   8.00g
	vm-1000-disk-0           pve Vwi-aotz--  70.00g data
	vm-1000-disk-1           pve Vwi-aotz--   4.00m data
  
(More volumes are listed but are not important here). All these volumes are mappd to /dev/pve/(swap, root) and also to /dev/mapper/(pve-root,pve-swap,pve-vm....)

The output is a bit tricky to understand. Basically, there is a pool called "data" which contains all the volumes for my VMs and containers ("t" - thin provisioned). Plus there is an 82 GB root partition and an 8 GB swap partition. As I wanted to be able to go back after the upgrade if things would go wrong, here's the idea:

## The Plan

1. Turn off swap usage
2. Convert the swap volume to a bootable partition
3. Boot into the (former) swap partition
4. shrink the root volume to roughly 50% (41 GB)
5. create a new root2 volume
6. copy all the data from the root volume to the new root2 volume
7. boot into root2 and do the upgrade in the new root2 volume
8. If things go wrong, revert back to the old root volume

## Step 1: Converting the swap volume into a bootable Linux

The swap volume is listed in your `/etc/fstab` - just comment out the line

	vi /etc/fstab
	# /dev/pve/swap none swap sw 0 0

Then we switch off swapping on the running system, convert the swap volume into a file system and create a copy of the running linux to the new file system:

	# check if swap is used
	swapon -v
	# switch swap off
	swapoff
	# remove the entry from fstab
	vi /etc/fstab
	# see if anything else is mounted
	mount -a
	swapon -v
	# create an ext4 file system on the former swap volume
	# and mount it in /mnt/swap
	mke2fs -j /dev/mapper/pve-swap
	mkdir /mnt/swap
	mount -t ext4 /dev/mapper/pve-swap /mnt/swap

before we can rsync the files over to the (former) swap volume, we need to stop all services that have files open (I forgot to stop the firewall service but the copy was still "good enough" to temporarily boot from it)

	# check for running services and stop them
	systemctl status
	systemctl stop lxc-monitord.service
	systemctl stop lxcfs.service
	systemctl stop pve-lxc-syscalld.service
	# now rsync the files over from root to the former swap
	# we need to exclude pseudo file systems like dev, sys, proc and the like
	rsync -aAXv / --exclude={"/dev/*","/proc/*","/sys/*","/tmp/*","/run/*","/mnt/*","/media/*","/lost+found"} /mnt/swap/
	# now rsync again

Now we have a good enough copy on the former swap volume - next we need to create a new EFI boot entry.

## Step 2: Copy grub and change the UEFI boot order

	# the grub entries for pve are in /boot/efi/EFI
	# we just copy the proxmox entry to a new entry called "pve6"
	cd /boot/efi/EFI
	mkdir pve6
	cp proxmox/grubx64.efi pve6/

	# now check the UEFI boot entries
	efibootmgr -v

	# see the partition that grubx64.efi would boot from
	strings proxmox/grubx64.efi 

	# create a new UEFI bios boot entry for the new one
	# please note the double back slashes!
	efibootmgr --create --disk /dev/sdb --part 2 --label "pve6" --loader \\EFI\\pve6\\grubx64.efi

	# change the boot order
	efibootmgr -o 0000,0003,0001,0002

	# bind mount dev etc.
	mount /dev/sdb2 /mnt/swap/boot/efi
	for i in /dev /dev/pts /proc /sys /run; do sudo mount -B $i /mnt/swap$i; done

	# chroot into the new root
	chroot /mnt/swap
	
	# now mount boot/efi and change the Grubx64.efi
	mount /dev/sdb2 /boot/efi
	grub-install --target=x86_64-efi /dev/sdb2

	# last but not least update the grub config
	update-grub

	# exit, unmount and reboot
	exit
	for i in /dev /dev/pts /proc /sys /run; do sudo umount  /mnt/swap$i; done
	reboot

Phew! That's it. The system should now boot into the (old) swap volume. We can now take an offline copy of the old system onto a second disk (/mnt/disk2) - because the next steps (shrinking the file system etc.) are risky.

## Step 3: Take a safety copy of the old system


	# system boots into /dev/mapper/pve-swap
	# now take another offline copy 
	mount -t ext4 /dev/mapper/pve-root /mnt/pveroot/
	mkdir /mnt/disk2/safetycopy
	rsync -aAXv /mnt/pveroot/ /mnt/disk2/safetycopy/

## Step 4: Shrink the original file system and volume:

	# unmount, check the file system and shrink it
	umount /mnt/pveroot/
	e2fsck -f /dev/mapper/pve-root 
	resize2fs /dev/mapper/pve-root 40G

	# now reduce the Logical Volume
	lvreduce -L 41G /dev/pve/root

	# mount and check
	mount -t ext4 /dev/mapper/pve-root /mnt/pveroot/
	ls -al /mnt/pveroot/

## Step 5: Create a new Volume ("root2") for pve7 and copy the pve6 volume over

	# looks good, create the new volume for pve 7
	lvcreate -n root2 -L41G /dev/pve

	# check it with lvs
	lvs
	# the "o" attribute shows it's open so let's unmount the old one and check again
	umount /dev/mapper/pve-root
	lvs

	# Now create the file system on the the new one, mount it and offline rsync everything over
	mke2fs -j /dev/mapper/pve-root2
	mkdir /mnt/pveroot2
	mount -t ext4 /dev/mapper/pve-root2 /mnt/pveroot2/
	rsync -aAXv /mnt/pveroot/ /mnt/pveroot2/

# Step 6: same story as before - copy efi, update uefi, grub-install and update-grub

	efibootmgr -v
	cd /boot/efi/EFI/
	cp -av proxmox/ swap
	ls -al
	efibootmgr --create --disk /dev/sdb --part 2 --label "swap" --loader \\EFI\\swap\\grubx64.efi
	efibootmgr -o 0000,0003,0004,0001,0002
	for i in /dev /dev/pts /proc /sys /run; do sudo mount -B $i /mnt/pveroot2$i; done
	chroot /mnt/pveroot2
	mount /dev/sdb2 /boot/efi
	grub-install --target=x86_64-efi /dev/sdb2
	update-grub

	# don't foregt to modify the new fstab
	vi /etc/fstab
	exit
	for i in /dev /dev/pts /proc /sys /run; do sudo umount  /mnt/pveroot2$i; done
	reboot

# Step 7: boot into root2 and do the in-place upgrade of Proxmox to V7

	pve6to7
	apt update
	apt dist-upgrade
	sed -i -e 's/buster/bullseye/g' /etc/apt/sources.list
	apt update
	apt dist-upgrade
	reboot


# Step 8: Revert to Version 6

Once I realized that Version 7 did not work as expected, I reverted to Version 6 simply by changing the boot order back to the "pve6" entry:

	efibootmgr -v
	efibootmgr -o 0003,0000,0004,0001,0002
	reboot


<a href="https://www.youtube.com/watch?v={{page.videoid}}"><img src="/assets/images/thumbnails/{{page.videoid}}.jpg">Watch the video on YouTube</a>

<details>
	<summary>Click to view the entire transcript</summary>
In this video I want to show you how I used the GRUB boot loader and the EFI bios of my Proxmox server in order to upgrade Proxmox from Version 6 to Version 7 and how I went back in just a minute. This video is all Linux command line. Proxmox is just a debian Linux that runs additional software such as LXC,KVM, QEMU and the like. What I’m showing you today is how to pivot from one Linux installation to another by using a temporary linux installation and shrinking the size of the original filesystem.

( The scenario )

Here’s the volume layout of my Proxmox Server system which is actually a PC which is located in a closet downstairs in the basement. There’s a root volume with Proxmox 6 in it and a swap volume. I will turn that little swap volume into a bootable Linux volume, boot into it with GRUB, shrink the original file system and volume, then add a second root volume, copy the first volume onto it, then I’ll instruct the BIOS to EFI boot from the second volume and I will then run the Proxmox upgrade there. After the upgrade I’ll figure out that some things are not working as expected. And as I have kept the original volume, I will revert the system back to Version 6 using GRUB in less than a minute. And the best: I will do all this remotely on a real headless system – even though I will reboot the server three or four times from different volumes, I will not enter the bios a single time. I have no screen or keyboard attached to that system. I’ll do everything from Linux over ssh. Hooray for command-line!

(Things to consider before an upgrade )

When you have a server and you need to upgrade it then you want to take care of a couple of things. If you are the only user of the server and of the services that it provides, then you don’t really care if things go wrong – after all, you are the only person that is affected. On the other side – if there are other people using that server then you want to take two basic things into consideration. One is downtime (in other words the time that the services will be offline while you do the upgrade) and the other one is the way back – the Plan B if things go wrong. Today we will not talk too much about downtime. With Proxmox you could have a second server or – even better – a cluster of three or more nodes and migrate the containers and VMs over to another node before you do the upgrade in order to minimize down time. I really want to focus on the Plan B scenario today.

Oh – and just want to add one thing at this point. I will be typing a lot of commands in the command line in this video. But please do not focus too much on those commands. I will provide a written blog version of all this on my blog on www.onemarcfifty.com. So if you want to go through this in detail later or copy paste commands then you can do it from there. The link to my blog is in the description of the video.

(Step 1 : Clean up of the old system)

Here’s the LVM disk layout of my system again. This is actually what Proxmox installs as a default if you only have one disk. It’s got a fast SSD that has three partitions. One for non-EFI boot (that’s the old school Master boot record or MBR boot for 32 bit systems), one for 64 bit EFI boot and one with the real data on it. We will only look into UEFI today. I think MBR is gone – thankfully. This third partition is the only disk in a Volume Group called pve that has basically the following Volumes and pools on it. There is my Linux root volume, there is a small swap volume and then there is a thin provisioned pool called data that in turn contains  all the volumes of the containers and VMs that I am running on that system. The root volume is roughly 100 GB in size. However – once I cleaned it up – that means – once I removed all the downloads, ISO images etc. from it – it turned out that it really only contained 4 to 5 GB of real Linux system data. And that actually led me to this crazy idea to use that Linux swap volume here as a kind of pivot installation. We’ll see in a minute what I mean by that.

Before I started of course I stopped all containers and VMs running on that Proxmox node. I should also have stopped and disabled a couple of services. But more on this later. Now lets see how we can turn a swap volume into a bootable Linux.

(Step 2 – Create a temporary boot installation on the Swap Volume)

A swap volume or partition can be switched on or off using the swapon and swapoff command in linux.  Swapon -v shows the currently used volumes and swapoff switches them off. The system is now not using them any more. I can now just convert it into a “normal” Linux ext4 file system using the mke2fs command – That creates an ext2, ext3 or ext4 file system inside the volume. I can now mount this new empty file system using the mount command. Just need to create a directory somewhere and then I can mount it there. So my plan is to boot from this. Therefore I need a Linux on it. What I want to do is just take a copy of the currently running Linux system and then boot from that copy by modifying the settings for GRUB. In order to take a copy I am using the rsync command. This copies over all the files, directories, links and so on from one volume to another. Just – if we do that we need to exclude a couple of things – the so called pseudo file systems. Those are file systems that only exist during runtime and are not physically present on the disc. Such as sys, proc, run and a couple of others. I got some errors during the copy process. Basically I don’t care too much because I will only use this volume temporarily. Nevertheless I figured out that the files which it couldn’t copy were related to LXC so I stopped all LXC related services and launched the rsync again. I should have taken the firewall down as well because that had some files in use as well.

On that note – taking a good copy of a running system is not that trivial. Basically you need to handle two things here. Files that are locked by processes and pseudo file systems that are mounted by processes. You can see the running processes by typing systemctl status and you can list open files by using the lsof command. In order to make this copy as good as possible I should have stopped all services on the machine first. But again – I am not planning to run Proxmox from that volume. I just need it temporarily.

(Step 3 – instruct the BIOS to EFI boot GRUB from the new system)

Cool. So now I have a kind of OK copy which I am quite confident that I can boot from. But how can I tell the BIOS to actually boot from that former swap volume and not from the pve root volume? Let’s have a look at how UEFI Bioses boot Linux. The EFI BIOS would first check the disks for an EFI partition. That’s actually a rather small partition, typically 100 or 200 MB in size that contains a FAT16 file system – the old MS DOS style file system, right?. Inside that file System we have a folder called EFI. Inside that folder typically you have subdirectories for each Operating system or Linux distribution or boot loader manager that is installed on the system, so you might have  Windows, Ubuntu, Debian, Proxmox, Clover, Apple and so on. And in each of these subdirectories there are x64 efi binaries which the bios then executes. You can do that from an EFI shell as well. Those binaries then load necessary drivers in order to be able to access an NTFS or HPFS+ or ext4 file system, depending on the OS and then try to boot the OS from that volume. But which one should it boot ? Apple, Ubuntu or Windows ? In the bios typically you can chose one to be the default boot loader. We will do this from Linux without being in front of the machine and hitting DEL or F2 or whatever at boot time. Here is how.

Under a Linux that has been booted using EFI, the EFI boot partition is mounted in /boot/efi. In my case that points to /dev/sdb2. Here in the EFI subdirectory we can find the binaries which the UEFI Bios will call in order to boot the system. That could be a Windows Boot Manager or Clover or in my case Grub. Let me take a copy of it into a separate subdirectory. Now we use the efibootmgr command to visualize the BIOS boot entries. And – we can also use it to add a new entry and to change the boot order. In my case I add the copy of the grubx64.efi which I have just taken as a new entry which I labeled pve6. My boot disk is /dev/sdb and the EFI partition is partition number 2. By default the newly added entry boots first, but I just want to keep that as a fall back solution and actually modify the original one which should then point to the former swap partition. I do this by launching the efibootmgr with the -o option. One reason why you can find so many reports on the internet of Linux installations going FUBAR after they have been moved from one disk or system to another is that the location of the partition where grub will look for the next step in the boot process is actually stored inside that grubx64.efi file. The next step in fact for the grub boot loader would be to load the config file that tells it which menu options to show – that’s the boot manager portion of grub. From there it would then load the initial ram disk where it can actually boot a basic Linux from. In order to change that entry or rather to make grubx64.efi point to another volume we need to use the program grub-install. Unfortunately grub-install needs to be ran from the volume that you want to boot to. In Linux we can use a little trick for that. We can actually change the root of the running installation by doing a chroot. But before we can chroot, we need to mount those pseudo file systems that I have been talking about. At least we need dev, proc, sys and run. We can mount these as so called bind mounts in the new root with the -B option and actually use the ones from the real system. Now I can chroot. I am now in the new root in a new Linux and can now install another version of the Grub loader onto the EFI partition. Just need to mount the efi partition inside the chroot cage before I can do that. As I want to make sure that Grub gets installed as EFI loader, I specify the target=x86_64-efi option here. Now I have taken care of the first two steps – I have a bios entry that loads the right boot manager and that boot manager points to the right volume. Now we need to look after the third step in that boot process – that’s the grub config files and those initramfs images here. That’s done by launching update-grub. So again – we need to do both – grub-install and update-grub. Once grub has mounted one of the initramfs images and loaded the kernel from there, it will then pivot the root to the volume that is specified in the config file and launch the /sbin/init process there. That init or systemd process will then do stuff like start services and mount the file systems and so on. So before we reboot we also need to make sure that the last step after the boot process is OK, that’s actually the /etc/fstab file which tells the new init process which file systems it needs to mount. We also need that to point to the new location.

So – to sum up – there are five locations or pointers that you need to change in order to EFI boot to a new partition or volume. The bios entry, the grub efi boot loader, then the grub boot manager config files, the initramfs images and last but not least the fstab file. Cool – now we should be all set to reboot the system into the new Linux on the former swap partition. Let’s reboot.

(Why I did it the way I did)

At this point – you might wonder why I did not simply boot from a Linux CD like Knoppix or the like. There’s actually four reasons for that. First – this machine has a quite complex network configuration with tagged VLANs. If I booted from a CD, then I would have had to spend a lot of time to get network connectivity back. Second – I had to chroot in order to reinstall GRUB. Doing a chroot is quite simple if the chroot Linux is similar or identical to the base Linux that you are running but can be a challenge if the versions of glibc or libc etc. are different. So using the same Linux makes this much easier. Third – as Proxmox installs everything into LVM volumes by default, I was sure that if I used the same Linux for emergency boot then I would have all the volumes readily mounted. But the decisive argument was the fourth reason. My Proxmox Server is headless. That means there is no screen or keyboard. Plus it’s in the basement. Somewhere between the ironing board and the washing machine. I would therefore have had to connect a Screen and Keyboard first and stand in front of the server. Not very nice. And guys – even though I am showing everything with LVM – it would work exactly the same with disk partitions. 

(Step 4 – Take an offline copy and shrink the original Volume)

The system has now rebooted into the new volume which I can actually verify by typing mount without any arguments and here I see that the swap volume is my current root. The next thing that I want to do is take an offline copy of the original Linux. The copy which I had made was just good enough to boot from it – it is not consistent in order to run Proxmox. So what I do is that I mount the old Proxmox 6 in /mnt/pveroot and rsync the content into a subdirectory on a second disk. I do not need to exclude the pseudo file systems because they are not mounted as that Linux system is not running. Why do I take a copy at all? Because the next step is going to be risky. I will now shrink the file system inside the original volume and then shrink the volume itself. I have shrinked the file system to 40 GB and I am adding a bit of safety margin to the volume by shrinking it to 41 GB. Just mounting it quickly in order to have a quick check if everything looks OK. So far so good.

(Step 5 – Create a new Volume and boot from it)

Now I have the free space to create a second volume which I call root2. I do this with lvcreate – the name is root2, the size is 41 GB and the volume group I create it in is pve. If I check with lvs then I can see both logical volumes. The o here just indicates that the original root is open. So let me unmount it and here we go – the o goes away. Now I do the same like I did with the swap partition in the beginning. Create a file system with mke2fs, mount it, rsync the files from the original root over. Take a copy of the efi boot loader, label the original one “swap” and create an efi boot entry for it. Change the boot order. Chroot into the new volume, grub-install, update-grub and modify /etc/fstab. Unmount proc,sys, run and reboot. 

(Step 6 – Run the upgrade)

At this point I have completed all preparation steps and I can now run the Proxmox upgrade in this partition. Proxmox comes with a program called pve6to7 that does a lot of checks if your environment is ready for the upgrade. All I have to do then is apt update, apt dist-upgrade, replace all occurences of buster with bullseye in the apt sources and do the dist-upgrade again. Reboot. 

(Step 7 – fail back to the old volume)

Let me check the Proxmox Interface – and yes, here it shows that I am now on version 7. I told you in the beginning that I had to revert back – but why ? The reason is that one of the machines that I am running on this Proxmox actually has the GPU passed through. And that pass through did not work as expected in Proxmox 7 at the time. If you search on the internet then you’ll find a lot of info about things working on some kernel versions and not on others. So I decided that I would not pursue this for the time being but rather revert back to version 6. And this is where the beauty of all this happens. I have spent a lot of time preparing this upgrade and the plan B, the fall back plan and so on. But as I have done this before, it now only takes me a minute to revert back. All I have to do is change the boot order in the efi bios with efibootmgr and reboot the machine. Bam. I am back on 6. And as I now have the second volume ready, I can retry this at a later point without the stress of going through all this. I just have to change the boot order back in order to check back if things have been fixed in Proxmox 7 or rather the underlying Kernel.
(Step 8 – clean up)

Last step of course- I should now turn the swap volume back into being swap and also fix the fstab files. This can be done using the mkswap command which will overwrite the ext4 file system on the volume.

Cool – that’s all I wanted to show you guys today. Please do leave me a comment on YouTube if you liked this episode and if it was useful for you. A like on YouTube is always appreciated of course. Having said that – many thanks for watching. Stay safe, stay healthy, bye for now.



</details>
