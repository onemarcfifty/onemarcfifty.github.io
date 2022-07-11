---
title: "Windows 11 with Hyper-V on Proxmox"
last_modified_at: 2022-06-22T17:00:00
categories:
  - Blog
tags:
  - Proxmox
  - Windows 11
  - Hyper-V
toc: true
header:
    teaser: /assets/images/blogimages/Win11_WSL2_on_proxmox.jpg
---

## How to install Windows 11 without TPM on Proxmox

The Windows 11 installer by default needs the "Trusted Platform Module" (TPM) and Secure Boot enabled. In a virtual environment such as KVM / QEMU (like we have it in Proxmox) those are not available. (Update - on Proxmox 7 you can enable it, on Proxmox 6 you can't) However, you can circumvent the TPM and Secure Boot checks by doing the following:

When the installation starts, you can type Shift-F10 in order to get to a command prompt. Inside that command prompt you can launch the registry editor by typing `regedt32` or `regedit` - now add the following key:

`HKEY_LOCAL_MACHINE\SYSTEM\Setup\LabConfig`

Under that key you create two DWORD32 values which you need to set to 1:

    BypassTPMCheck
    BypassSecureBootCheck

The complete tree should thus look as follows:


    [HKEY_LOCAL_MACHINE\SYSTEM\Setup\LabConfig]
    "BypassTPMCheck"=dword:00000001
    "BypassSecureBootCheck"=dword:00000001

And there you go - you can install Windows 11 either in a virtual environment such as KVM/QEMU or on older, unsupported hardware.

## How to run Hyper-V on Proxmox

**WARNING! This is highly experimental !!! See the Side effects section below**

If you want to enable let's say WSL2 in your Windows 11 VM then you need to enable Hyper-V. Hyper-V is a type 1 hypervisor, i.e. it expects to run on physical hardware. If you just create a VM on Proxmox without further customization, then the task manager inside Windows 11 would show the CPU as KVM CPU and refuse to start. The paravirtualization features of Hyper-V actually needs a bunch of features from the CPU which are not passed on to the VM which are called enlightenments (google for hyper-v enlightenments). The trick is now to tell KVM/QEMU that it should not present a KVM CPU to the VM but rather the hosts CPU and enable those enlightenments.

I did this on my AMD Ryzen9 with the following parameters (you might not need all of them plus they might look different on Intel hardware):

    args: -cpu host,hv_relaxed,hv_vapic,hv_spinlocks=0x1fff,hv_vpindex,hv_runtime,hv_crash,hv_time,hv_synic,hv_stimer,hv_tlbflush,hv_ipi,hv_reset,hv_frequencies,hv_reenlightenment,hv_stimer_direct,hv-no-nonarch-coresharing=auto

The (nearly) full config file `/etc/pve/qemu-server/xxx.conf` (replace xxx with the ID of your VM) looks like this:

    args: -cpu host,hv_relaxed,hv_vapic,hv_spinlocks=0x1fff,hv_vpindex,hv_runtime,hv_crash,hv_time,hv_synic,hv_stimer,hv_tlbflush,hv_ipi,hv_reset,hv_frequencies,hv_reenlightenment,hv_stimer_direct,hv-no-nonarch-coresharing=auto
    bios: ovmf
    boot: order=sata0;virtio0
    cores: 12
    # I used the virtio drivers for the harddisk controller
    sata2: local-disc:iso/virtio-win-0.1.217.iso,media=cdrom,size=519172K
    scsihw: virtio-scsi-pci
    # replace xxx with the machine number
    efidisk0: yourstorage:xxx/vm-xxx-disk-1.qcow2,size=128K
    virtio0: yourstorage:xxx/vm-xxx-disk-0.qcow2,cache=writeback,size=200G
    machine: q35
    memory: 16384
    name: win11-proxmox
    # put in the right bridge
    net0: e1000=12:34:56:78:90:AB,bridge=vmbrxx,firewall=1
    numa: 0
    ostype: l26
    sata0: none,media=cdrom
    smbios1: uuid=YOUR-UU-ID
    sockets: 1
    vga: std
    vmgenid: UUID-OF-YOUR-VM

Windows 11 now correctly reports the CPU in task manager:

![Windows 11 running inside KVM on Proxmox](/assets/images/blogimages/2022-06-07-hyperv-taskmgr.jpg)

## Side effects

My primary goal was to run WSL2 for testing. It runs fine inside Windows 11. All you have to do is install it with `wsl --install -d debian` for example.

As we run the hypervisor on top of a virtualized environment, the machine is roughly 25% slower (as the virtualization layer below needs to emulate the hardware) - ~~besides that, I have seen no side effects so far.~~ ~~**After the 2nd reboot I ran into issues~~

~~I had been working intensively with snapshots, therefore I initially did not realize that there is a big problem. After the 2nd reboot this did not work any more. After 10 minutes or so the machine went to high CPU utilization (100%) and became unusable. Checking the forums it turns out that this is a known issue. Things seem to be fixed with Kernel version 5.15 - I will update here as soon as I have more info. For the time being this does not work on AMD Ryzen~~

## All working in Kernel 5.15

After an upgrade of the Proxmox Kernel to Version 5.15 everything is working like a charm!

    root@pve2:~# uname -r
    5.15.35-2-pve

The issues I had with GPU passthrough could be resolved by adding the following to the `/etc/default/grub` file:

    GRUB_CMDLINE_LINUX_DEFAULT="quiet amd_iommu=on iommu=pt video=vesafb:off video=efifb:off initcall_blacklist=sysfb_init"

The key component here is the `initcall_blacklist=sysfb_init` statement in order to prevent bootfb from holding on to the GPU