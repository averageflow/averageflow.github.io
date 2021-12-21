---
layout: post
title: Arch on UEFI with Encryption
date: 2021-12-21
description: Installing Arch Linux on UEFI with Full Disk Encryption
categories: ["Arch", "Linux", "UEFI", "installation", "encryption"]
---

# Installing Arch Linux on UEFI with Full Disk Encryption

This is a step by step guide to installing Arch Linux on UEFI with full disk encryption. It deliberately contains no unnecessary words or bling. It is havily based on the Arch Linux wiki’s installation guide so if you’re ever stuck, just refer to it and the rest of the awesome Arch wiki.

## Download ISO

    Download the latest ISO from the Arch Linux website

## Create Bootable USB Stick

You can skip this step if you just want to run Arch Linux in a VM. In that case, just run the ISO from your favorite VM management tool like QEMU.

    Insert an USB stick into your computer
    Run lsblk to find the correct disk
    Run sudo umount /dev/sdx or whatever the USB stick is
    Run sudo dd bs=4M if=path/to/input.iso of=/dev/sdx oflag=sync status=progress to write the ISO to the USB stick. Don’t forget to replace the two paths with the correct ones.
    Insert the USB stick into the target computer and boot it from there

As soon as you can see the Arch Linux prompt, you are ready for the next step.

## Check for UEFI support

    Run ls /sys/firmware/efi/efivars to check if that directory exists. If it doesn’t, your system does not support UEFI and this guide is not for you and you should refer to the official Arch Linux Installation Guide instead.

## Establish Connectivity

    Connect the computer via ethernet (recommended) or run iwctl to log into WiFi
    Check for internet connectivity with ping archlinux.org
    Make sure the clock is synced with timedatectl set-ntp true

## Partition

    Check for different drives and partitions with lsblk and then start to partition with gdisk /dev/nvme0n1 (or whatever the disk is)
    Delete any existing partitions using d
    Create boot partition with n with default number, default first sector, last sector at +512M and select ef00 “EFI System” as the type
    Create root partition with n with default number, default first sector, default last sector and select 8300 “Linux filesystem” as the type
    Press w to write partitions
    Run lsblk again to verify partitioning

## Encrypt Root Partition

    Run cryptsetup -y -v luksFormat /dev/nvme0n1p2 and then type YES and the new encryption password to encrypt the root partition
    Run cryptsetup open /dev/nvme0n1p2 cryptroot to open the encrypted partition

## Create File Systems

    Create the boot file system with mkfs.fat -F32 /dev/nvme0n1p1 (or whatever the partition is called)
    Create the root file system with mkfs.ext4 /dev/mapper/cryptroot

## Mount File Systems

    Run mount /dev/mapper/cryptroot /mnt to mount the root file system
    Run mkdir /mnt/boot to create the boot directory
    Run mount /dev/nvme0n1p1 /mnt/boot to mount your boot file system
    Run lsblk again to verify mounting

## Create Swap File (not needed on VMs)

    Run dd if=/dev/zero of=/mnt/swapfile bs=1M count=24576 status=progress to create the swap file where the count is the number of mebibytes you want the swap file to be (usually around 1.5 times the size of your RAM)
    Run chmod 600 /mnt/swapfile to set the right permissions on it
    Run mkswap /mnt/swapfile to make it an actual swap file
    Run swapon /mnt/swapfile to turn it on

## Install Arch Linux

    Run pacstrap /mnt base base-devel linux linux-firmware neovim to install Arch Linux (linux-firmware is not needed on VMs)

## Generate File System Table

    Run genfstab -U /mnt >> /mnt/etc/fstab to generate fstab with UUIDs

## Switch to Your New Linux Installation

    Run arch-chroot /mnt to switch to your new Arch Linux installation

## Set Locales

    Run ln -sf /usr/share/zoneinfo/Europe/Zurich /etc/localtime (or whatever your timezone is) to set your time zone
    Run hwclock --systohc
    Run nvim /etc/locale.gen and uncomment yours (e.g. en_US.UTF-8 UTF-8)
    Run locale-gen to generate the locales
    Run echo 'LANG=en_US.UTF-8' > /etc/locale.conf

## Set Hostname

    Run echo 'arch' > /etc/hostname (or whatever your hostname should be)
    Run nvim /etc/hosts and insert the following lines:

127.0.0.1     localhost
::1           localhost
127.0.1.1     arch.localdomain        arch

for the last line: change arch to whatever hostname you picked in the last step

## Set Root Password

    Run passwd and set your root password

## Configure Initramfs

    Run nvim /etc/mkinitcpio.conf and, to the HOOKS array, add keyboard between autodetect and modconf and add encrypt between block and filesystems
    Run mkinitcpio -P

## Install Boot Loader

    Run pacman -S grub efibootmgr intel-ucode (or amd-ucode if you have an AMD processor) to install the GRUB package and CPU microcode
    Run blkid -s UUID -o value /dev/nvme0n1p2 to get the UUID of the device
    Run nvim /etc/default/grub and set GRUB_TIMEOUT=0 to disable GRUB waiting until it chooses your OS (only makes sense if you don’t dual boot with another OS), then set GRUB_CMDLINE_LINUX="cryptdevice=UUID=xxxx:cryptroot" while replacing “xxxx” with the UUID of the nvme0n1p2 device to tell GRUB about our encrypted file system
    Run grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB to install GRUB for your system
    Run grub-mkconfig -o /boot/grub/grub.cfg to configure GRUB

## Install Network Manager

    Run pacman -S networkmanager to install NetworkManager
    Run systemctl enable NetworkManager to run NetworkManager at boot

## Reboot

    Run exit to return to the outer shell
    Run reboot to get out of the setup

## Connect to WiFi (only needed if there’s no ethernet connection)

    Run nmcli d wifi list to list the available networks
    Run nmcli d wifi connect MY_WIFI password MY_PASSWORD to connect to one of them

## Add User

    Run EDITOR=nvim visudo and uncomment %wheel ALL=(ALL) NOPASSWD: ALL to allow members of the wheel group to run privileged commands
    Run useradd --create-home --groups wheel,video lena (or whatever your user name should be) to create the user
    Run passwd lena to set your password
    Run exit and log back in with your new user

## Install Window Manager

    Run sudo pacman -S sway swayidle swaylock to install Sway
    Add the following to ~/.zlogin or whatever shell you are using:

## Start window manager

if [ "$(tty)" = "/dev/tty1" ]; then
  exec sway
fi

## Set Up Sound

    Run sudo pacman -S pipewire pipewire-pulse to install Pipewire

## Set Up Bluetooth

    Run sudo pacman -S bluez bluez-utils to install the bluetooth utilities
    Run sudo systemctl enable bluetooth.service --now to start bluetooth

## Lock Root User (to be extra secure)

    Run sudo passwd -l root to lock out the root user

## Install a Firewall

    Run sudo pacman -S nftables to install the firewall
    Run sudo nvim /etc/nftables.conf to edit the config to our liking and remove the part about allowing incoming SSH connections if you don’t need that
    Run sudo systemctl enable nftables.service --now to enable the firewall

## Enable Time Synchronization

    Run sudo systemctl enable systemd-timesyncd.service --now to enable automated time synchronization

## Improve Power Management (only makes sense on laptops)

    Run sudo pacman -S tlp tlp-rdw to install TLP
    Run sudo systemctl enable tlp.service --now to run power optimizations automatically
    Run sudo systemctl enable NetworkManager-dispatcher.service --now to prevent conflicts
    Run sudo tlp-stat and follow any recommendations there

## Enable Scheduled fstrim (only makes sense for SSDs)

    Run sudo systemctl enable fstrim.timer --now to enable regular housekeeping of your SSD

## Enable Scheduled Mirrorlist Updates

    Run sudo pacman -S reflector to install reflector
    Run sudo nvim /etc/xdg/reflector/reflector.conf and change the file to your liking
    Run sudo systemctl enable reflector.timer --now to enable running reflector regularly

## Reduce Swappiness (only makes sense if you have more than 4GB of RAM)

    Run echo 'vm.swappiness=10' | sudo tee /etc/sysctl.d/99-swappiness.conf to reduce the swappiness permanently

## Install Dotfiles

    Run sudo pacman -S git to install Git
    Install Cloudlena’s dotfiles or some other ones to customize your installation

# Arch + i3-gaps Install Guide
First set up your keyboard layout. For example, in Spanish:
```
   # loadkeys es
```
For a list of all acceptable keymaps:
```
   # localectl list-keymaps
```

## Pre-installation

### EFI-enabled BIOS
This guide assumes you have EFI-enabled BIOS. Let's check it out.
```
   # ls /sys/firmware/efi/efivars
```
When you run this command you should see a list of files.

### Internet connection
To make sure you have an internet connection, you have to ask Mr. Google:
```
   # ping 8.8.8.8
```

### Add best Arch mirrors
To install arch you have to download packages. It's a good idea to download
them from the best connection mirror.
```
   # pacman -Sy
   # pacman -S reflector
   # reflector --latest 5 --sort rate --save /etc/pacman.d/mirrorlist
```

### Partition disk
Your primary disk will be known from now on as `sda`. You can check if
this is really your primary disk:
```
   # lsblk
```
Feel free to adapt the rest of the guide to `sdb` or any other if you
want to install Arch on a secondary hard drive.

This guide will use a 250GB hard disk and will have only Arch Linux installed.
You'll create 5 partitions of the disk (feel free to suit this to your needs).

* `/dev/sda1` boot partition (1G).
* `/dev/sda2` swap partition (4G).
* `/dev/sda3` root partition (50G).
* `/dev/sda4` home partition (100G).
* `/dev/sda5` data partition (remaining disk space).

You're going to start by removing all the previous partitions and creating
the new ones.
```
   # gdisk /dev/sda
```
This interactive CLI program allows you to enter commands for managing your HD.
I'm going to show you only the commands you need to enter.

#### Clear partitions table
```
   Command: O
   Y
```
#### EFI partition (boot)
```
   Command: N
   ENTER
   ENTER
   +1G
   EF00
```

#### SWAP partition
```
   Command: N
   ENTER
   ENTER
   +4G
   8200
```

#### Root partition (/)
```
   Command: N
   ENTER
   ENTER
   +50G
   8304
```

#### Home partition
```
   Command: N
   ENTER
   ENTER
   +100G
   8302
```

#### Data partition
```
   Command: N
   ENTER
   ENTER
   ENTER
   ENTER
```

#### Save changes and exit
```
   Command: W
   Y
```

### Format partitions
```
   # mkfs.fat -F32 /dev/sda1
   # mkswap /dev/sda2
   # mkfs.ext4 /dev/sda3
   # mkfs.ext4 /dev/sda4
   # mkfs.ext4 /dev/sda5
```

### Mount partitions
```
   # swapon /dev/sda2
   # mount /dev/sda3 /mnt
   # mkdir /mnt/{boot,home}
   # mount /dev/sda1 /mnt/boot
   # mount /dev/sda4 /mnt/home
```

If you run the `lsblk` command you should see something like this:
```
   NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
   sda      8:0    0 232.9G  0 disk
   ├─sda1   8:1    0     1G  0 part /mnt/boot
   ├─sda2   8:2    0     4G  0 part [SWAP]
   ├─sda3   8:3    0    50G  0 part /mnt
   ├─sda4   8:4    0   100G  0 part /mnt/home
   └─sda5   8:5    0  77.9G  0 part
```

## Installation

### Update the system clock
```
   # timedatectl set-ntp true
```

### Install Arch packages
```
   # pacstrap /mnt base base-devel openssh linux linux-firmware neovim
```

### Generate fstab file
```
   # genfstab -U /mnt >> /mnt/etc/fstab
```

## Add basic configuration

### Enter the new system
```
   # arch-chroot /mnt
```

### Language-related settings
```
   # nvim /etc/locale.gen
```
Now you have to uncomment the language of your choice, for example
`en_US.UTF-8 UTF-8`.
```
   # locale-gen
   # nvim /etc/locale.conf
```
Add this content to the file:
```
   LANG=en_US.UTF-8
   LANGUAGE=en_US
   LC_ALL=C
```
```
   # nvim /etc/vconsole.conf
```
Add this content to the file:
```
   KEYMAP=us
```

### Configure timezone
For this example I'll use "Europe/Madrid", but adapt it to your zone.
```
   # ln -sf /usr/share/zoneinfo/Europe/Madrid /etc/localtime
   # hwclock —-systohc
```

### Enable SSH, NetworkManager and DHCP
These services will be started automatically when the system boots up.
```
   # pacman -S dhcpcd networkmanager network-manager-applet
   # systemctl enable sshd
   # systemctl enable dhcpcd
   # systemctl enable NetworkManager
```

### Install bootloader
```
   # pacman -S grub-efi-x86_64 efibootmgr
   # grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=arch
   # grub-mkconfig -o /boot/grub/grub.cfg
```
You can replace "arch" with the id of your choice.

### Choose a name for your computer
Assuming your computer is known as "thinkpad":
```
   # echo thinkpad > /etc/hostname
```

### Adding content to the hosts file
```
   # nvim /etc/hosts
```
And add this content to the file:
```
   127.0.0.1    localhost.localdomain   localhost
   ::1          localhost.localdomain   localhost
   127.0.0.1    thinkpad.localdomain    thinkpad
```
Replace "thinkpad" with your computer name.

### Install other useful packages
```
   # pacman -S iw wpa_supplicant dialog intel-ucode git reflector lshw unzip htop
   # pacman -S wget pulseaudio alsa-utils alsa-plugins pavucontrol xdg-user-dirs
```

### Update root password
```
   # passwd
```

### Final steps
```
   # exit
   # umount -R /mnt
   # swapoff /dev/sda2
   # reboot
```

## Post-install configuration

Now your computer has restarted and in the login window on the tty1 console you
can log in with the root user and the password chosen in the previous step.

### Add your user
Assuming your chosen user is "thinkpad":
```
   # useradd -m -g users -G wheel,storage,power,audio thinkpad
   # passwd thinkpad
```

### Grant root access to our user
```
   # EDITOR=nvim visudo
```
If you prefer not to be prompted for a password every time you run a command
with "sudo" privileges you need to uncomment this line:
```
   %wheel ALL=(ALL) NOPASSWD: ALL
```
Or if you prefer the standard behavior of most Linux distros you need to
uncomment this line:
```
   %wheel ALL=(ALL) ALL
```

### Login into newly created user
```
   # su - thinkpad
   $ xdg-user-dirs-update
```

### Install AUR package manager
In this guide we'll install [yay](https://github.com/Jguer/yay) as the
AUR package manager. More about [AUR](https://aur.archlinux.org/).

TL;DR AUR is a Community-driven package repository.
```
   $ mkdir Sources
   $ cd Sources
   $ git clone https://aur.archlinux.org/yay.git
   $ cd yay
   $ makepkg -si
```

### The coolest Pacman
If you want to make Pacman look cooler you can edit the configuration file and
uncomment the `Color` option and add just below the `ILoveCandy` option.
```
   $ sudo nvim /etc/pacman.conf
```

### PulseAudio applet
If you want to manage your computer's volume from a small icon in the systray:
```
   $ yay -S pa-applet-git
```

### Manage Bluetooth
```
   $ sudo pacman -S bluez bluez-utils blueman
   $ sudo systemctl enable bluetooth
```

### Improve laptop battery consumption
```
   $ sudo pacman -S tlp tlp-rdw powertop acpi
   $ sudo systemctl enable tlp
   $ sudo systemctl enable tlp-sleep
   $ sudo systemctl mask systemd-rfkill.service
   $ sudo systemctl mask systemd-rfkill.socket
```
If your laptop is a ThinkPad, also run this:
```
   $ sudo pacman -S acpi_call
```

### Enable SSD TRIM
```
   $ sudo systemctl enable fstrim.timer
```

## i3-gaps related steps

### Install graphical environment and i3
```
   $ sudo pacman -S xorg-server xorg-apps xorg-xinit
   $ sudo pacman -S i3-gaps i3blocks i3lock numlockx
```

### Install display manager
The display manager allows us to log in to the system graphically and also
to automate the startup of some services.
[LightDM](https://wiki.archlinux.org/index.php/LightDM) is one of the most
lightweight display managers.
```
   $ sudo pacman -S lightdm lightdm-gtk-greeter --needed
   $ sudo systemctl enable lightdm
```

### Install some basic fonts
```
   $ sudo pacman -S noto-fonts ttf-ubuntu-font-family ttf-dejavu ttf-freefont
   $ sudo pacman -S ttf-liberation ttf-droid ttf-roboto terminus-font
```

### Install some useful tools on i3
```
   $ sudo pacman -S rxvt-unicode ranger rofi dmenu --needed
```

### Install some GUI programs
```
   $ sudo pacman -S firefox vlc --needed
```

### Apply previous settings
```
   $ sudo reboot
```

## Rice i3-gaps
"Rice" is how we know to make visual improvements and customizations
to the desktop and its programs.

### Console
The program we use the most is the console emulator. I use
[kitty](https://github.com/kovidgoyal/kitty) as a replacement for urxvt,
the default terminal emulator on i3. However, these improvements can be
applied to any terminal emulator.

#### zsh
`zsh` is an alternative to `bash` shell I particularly love.
You can also have a look at the `fish` shell, because even though I haven't
tried it, it looks very cool.
```
   $ sudo pacman -S zsh
```

#### Oh My Zsh
There is also a set of add-ons that we install on top of `zsh` that provide us
a series of interesting goodies, as well as a new and improved prompt.

To install [Oh My Zsh](https://ohmyz.sh) we need to run the following command:
```
   $ sh -c "$(wget https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh -O -)"
```
During the execution of the command you will be asked if you want to set `zsh`
as the default shell for your user. You have to confirm this.

### LXAppearance
With LXAppearance you can change themes, icons, cursors or fonts.
```
   $ sudo pacman -S lxappearance
```
Once you have installed LXAppearance, you can start exploring the many possible
customization options by installing the great
[Arc theme](https://github.com/arc-design/arc-theme) and the
[Papirus](https://github.com/PapirusDevelopmentTeam/papirus-icon-theme) icons
theme.
```
   $ sudo pacman -S arc-gtk-theme
   $ sudo pacman -S papirus-icon-theme
```

### Customize LightDM
At this point you can also customize the look of LigthDM. You can blow your mind
by adding Papirus icons and Arc theme in LightDM, just by editing its config file.

```
   $ sudo nvim /etc/lightdm/lightdm-gtk-greeter.conf
```
In this file you have to add these lines
```
   [greeter]
   theme-name = Arc-Dark
   icon-theme-name = Papirus-Dark
   background = #2f343f
```
You can also custom the font with the same one you added in LXAppearance,
just by adding `font-name = Whatever` to this file.
