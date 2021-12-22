---
layout: post
title: Arch Linux UEFI with Encryption
date: 2021-12-21
description: Installing Arch Linux on UEFI with Full Disk Encryption
categories: ["Arch", "Linux", "UEFI", "installation", "encryption"]
---

This is a step by step guide to installing Arch Linux on UEFI with full disk encryption. It deliberately contains no unnecessary words or bling. It is heavily based on the Arch Linux wiki’s installation guide so if you’re ever stuck, just refer to it and the rest of [the awesome Arch wiki](https://wiki.archlinux.org/).

### Download ISO

Download the latest ISO from [the Arch Linux website](https://archlinux.org/download/).

### Create Bootable USB Stick

You can skip this step if you just want to run Arch Linux in a VM. In that case, just run the ISO from your favorite VM management tool like QEMU, VirtualBox or VMWare.

Insert an USB stick into your computer and run `lsblk` to find the correct disk.

Then run `sudo umount /dev/sdx` or whatever your USB stick is named.

Run `sudo dd bs=4M if=path/to/input.iso of=/dev/sdx oflag=sync status=progress` to write the ISO to the USB stick. Don’t forget to replace the two paths with the correct ones.

Insert the USB stick into the target computer and boot it from there.

As soon as you can see the Arch Linux prompt, you are ready for the next step. If you don't know how to boot from a USB stick on a computer, you probably shouldn't go for Arch Linux and its "do-it yourself" attitude.

### Check for UEFI support

Run `ls /sys/firmware/efi/efivars` to check if that directory exists. 

If it doesn’t, your system does not support UEFI and this guide is not for you and you should refer to the official Arch Linux Installation Guide for Legacy BIOS / MBR instead.

### Establish Connectivity

Connect the computer via ethernet (recommended) or run `iwctl` to log into WiFi. Check for internet connectivity with `ping archlinux.org`. Once connected make sure the clock is synced with `timedatectl set-ntp true`.

### Partition

Check for different drives and partitions with `lsblk` and then start to partition with `gdisk /dev/nvme0n1` (or whatever the disk is).

Delete any existing partitions using `d`.

Create boot partition with `n` with default number, default first sector, last sector at `+512M` and select `ef00` “EFI System” as the type.

Create root partition with `n` with default number, default first sector, default last sector and select `8300` “Linux filesystem” as the type.

Press `w` to write partitions.

Run `lsblk` again to verify partitioning.

### Encrypt Root Partition

Run `cryptsetup -y -v luksFormat /dev/nvme0n1p2` and then type `YES` and the new encryption password to encrypt the root partition.

Run `cryptsetup open /dev/nvme0n1p2 cryptroot` to open the encrypted partition.

### Create File Systems

Create the boot file system with `mkfs.fat -F32 /dev/nvme0n1p1` (or whatever the partition is called).

Create the root file system with `mkfs.ext4 /dev/mapper/cryptroot`.

### Mount File Systems

Run `mount /dev/mapper/cryptroot /mnt` to mount the root file system.

Run `mkdir /mnt/boot` to create the boot directory.

Run `mount /dev/nvme0n1p1 /mnt/boot` to mount your boot file system.

Run `lsblk` again to verify mounting.

### Create Swap File (not needed on VMs)

Run `dd if=/dev/zero of=/mnt/swapfile bs=1M count=20576 status=progress` to create the swap file where the count is the number of megabytes you want the swap file to be (usually around 1.5 times the size of your RAM).

Run `chmod 600 /mnt/swapfile` to set the right permissions on it.

Run `mkswap /mnt/swapfile` to make it an actual swap file.

Run `swapon /mnt/swapfile` to turn it on.

### Install Arch Linux

Run `pacstrap /mnt base base-devel linux linux-firmware neovim` to install Arch Linux (linux-firmware is not needed on VMs).

### Generate File System Table

Run `genfstab -U /mnt >> /mnt/etc/fstab` to generate fstab with UUIDs.

### Switch to Your New Linux Installation

Run `arch-chroot /mnt` to switch to your new Arch Linux installation.

### Set Locales

Run `ln -sf /usr/share/zoneinfo/Europe/Amsterdam /etc/localtime` (or whatever your timezone is) to set your time zone.

Run `hwclock --systohc`.

Run `nvim /etc/locale.gen` and uncomment yours (e.g. en_US.UTF-8 UTF-8).

Run `locale-gen` to generate the locales.

Run `echo 'LANG=en_US.UTF-8' > /etc/locale.conf`.

### Set Hostname

Run `echo 'myHostname' > /etc/hostname` (or whatever your hostname should be).

Run `nvim /etc/hosts` and insert the following lines:

```
127.0.0.1     localhost
::1           localhost
127.0.1.1     arch.localdomain        myHostname
```

for the last line: change `myHostname` to whatever hostname you picked in the previous step.

### Set Root Password

Run `passwd` and set your root password.

### Configure Initramfs

Run `nvim /etc/mkinitcpio.conf` and, to the `HOOKS` array, add `keyboard` between `autodetect` and `modconf` and add `encrypt` between `block` and `filesystems`.

Run `mkinitcpio -P`.

### Install Boot Loader

Run `pacman -S grub efibootmgr intel-ucode` (or `amd-ucode` if you have an AMD processor) to install the GRUB package and CPU microcode.

Run `blkid -s UUID -o value /dev/nvme0n1p2` to get the UUID of the device.

Run `nvim /etc/default/grub` and set `GRUB_TIMEOUT=0` to disable GRUB waiting until it chooses your OS (only makes sense if you don’t dual boot with another OS), then set `GRUB_CMDLINE_LINUX="cryptdevice=UUID=xxxx:cryptroot"` while replacing `xxxx` with the UUID of the `nvme0n1p2` device to tell GRUB about our encrypted file system.

Run `grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB` to install GRUB for your system.

Run `grub-mkconfig -o /boot/grub/grub.cfg` to configure GRUB.

### Install Network Manager

Run `pacman -S networkmanager` to install NetworkManager.

Run `systemctl enable NetworkManager` to run NetworkManager at boot.

### Reboot

Run `exit` to return to the outer shell.

Run `reboot` to get out of the setup by restarting the machine. 

After the machine starts, remove the installation medium and you will boot into Arch Linux from your HDD.

### Connect to WiFi (only needed if there’s no ethernet connection)

The use of `nmcli` is prefered to `iwctl` for reasons of simplicity and user-friendliness.

Run `nmcli d wifi list` to list the available networks.

Run `nmcli d wifi connect MY_WIFI password MY_PASSWORD` to connect to one of them.

### Add User

Run `EDITOR=nvim visudo` and uncomment `%wheel ALL=(ALL) NOPASSWD: ALL` to allow members of the `wheel` group to run privileged commands.

Run `useradd --create-home --groups wheel,video joe` (or whatever your user name should be) to create the user.

Run `passwd joe` to set your password.

Run `exit` and log back in with your new user.


### Install a Firewall

Run `sudo pacman -S nftables` to install the firewall

Run `sudo nvim /etc/nftables.conf` to edit the config to our liking and remove the part about allowing incoming SSH connections if you don’t need that.

Run `sudo systemctl enable nftables.service --now` to enable the firewall.

### Enable Time Synchronization

Run `sudo systemctl enable systemd-timesyncd.service --now` to enable automated time synchronization.

### Improve Power Management (only makes sense on laptops)

Run `sudo pacman -S tlp tlp-rdw` to install TLP.

Run `sudo systemctl enable tlp.service --now` to run power optimizations automatically.

Run `sudo systemctl enable NetworkManager-dispatcher.service --now` to prevent conflicts.

Run `sudo tlp-stat` and follow any recommendations there.

### Enable Scheduled fstrim

This only makes sense for SSD.

Run `sudo systemctl enable fstrim.timer --now` to enable regular housekeeping of your SSD.

### Enable Scheduled Mirrorlist Updates

Run `sudo pacman -S reflector` to install reflector.

Run `sudo nvim /etc/xdg/reflector/reflector.conf` and change the file to your liking.

Run` sudo systemctl enable reflector.timer --now` to enable running reflector regularly.

### Reduce Swappiness

This only makes sense if you have more than 4GB of RAM.

Run `echo 'vm.swappiness=10' | sudo tee /etc/sysctl.d/99-swappiness.conf` to reduce the swappiness permanently.

### The coolest Pacman

If you want to make Pacman look cooler you can edit the configuration file with `sudo nvim /etc/pacman.conf` and uncomment the `Color` option and add just below the `ILoveCandy` option.

### Installing Plasma DE

I personally choose for Plasma DE (KDE), but of course if you prefer another setup now is the time to leave this guide. Hope you found it useful.

To install Plasma run `sudo pacman -S xorg plasma plasma-wayland-session kde-applications`. This might take a while.

Then enable auto-start at boot with
```
sudo systemctl enable sddm.service
sudo systemctl enable NetworkManager.service
```

And restart your machine with `sudo reboot`.

## Welcome to the master race of Arch Linux

 
 
