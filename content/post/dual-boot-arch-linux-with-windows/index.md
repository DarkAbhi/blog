+++
author = "Abhishek AN"
title = "Dual boot Arch Linux with Windows(UEFI)"
date = "2019-04-02"
summary = "Learn how to dual-boot Arch Linux with Windows on a UEFI machine using proper partitioning, GRUB setup, and configuration for a smooth and stable multi-OS system."
tags = [
    "Linux",
    "Arch Linux",
]
categories = [
    "Linux"
]
+++

If you have come here, you should have downloaded the Arch ISO and created a bootable disk and did the following:

  1. Disable Secure Boot
  2. Disable Launch CSM or Legacy Support
  3. Set Boot Mode to UEFI
  4. Enable USB Boot
  5. Set USB Disk as boot priority
   
Installing Arch Linux is a notably quite complicated procedure compared to other Linux/GNU distros. This guide is aimed for EFI systems which most modern computers come with. Let's begin the installation!

To verify if you are in UEFI mode, type
```html
# ls /sys/firmware/efi/efivars
```
## Create partitions using cgdisk

Type

```
cgdisk /dev/sda
```

Create the swap partition:

    Hit New again from the options at the bottom of partition list.
    Just hit enter to select the default option for the first sector.
    For the swap partition size, it is advisable to have 1.5 times the size of your RAM. I have 8GB of RAM so I’m gonna put 12GiB for the partition size. Hit enter.
    Set GUID to 8200. Hit enter.
    Set name to swap. Hit enter.

Create the root partition:

    Hit New again.
    Hit enter to select the default option for the first sector.
    Hit enter again to use the remainder of the disk.
    Also hit enter for the GUID to select default.
    Then set name of the partition to root.

Hit write at the bottom of the patitions list to write the changes to the disk. Type yes to confirm the write command. Now we are done partitioning the disk. Hit Quit to exit cgdisk.

See new partition table
```html
lsblk
```

## Output should be similar to this

<pre>
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda      8:0    0 238.5G  0 disk 
├─sda1   8:1    0   450M  0 part 
├─sda2   8:2    0   100M  0 part 
├─sda3   8:3    0    16M  0 part 
├─sda4   8:4    0  63.1G  0 part 
├─sda5   8:5    0   851M  0 part 
├─sda6   8:6    0  99.6G  0 part 
├─sda7   8:7    0    10G  0 part 
└─sda8   8:8    0  64.4G  0 part 
</pre>

Enable the swap partition:

```html
mkswap -L "Linux Swap" /dev/sda7
swapon /dev/sda7
```
To Verify swap is activated:

```html
free -m
```
Format root partition as ext4:

```html
mkfs.ext4 /dev/sda8
```
Mount the root partition:
```html
mkdir /mnt
mount /dev/sda8 /mnt
```

Connect to a network using Ethernet or Wirelessly using
```html
wifi-menu
```

## Install the base components

As of [2019-10-06](https://www.archlinux.org/news/base-group-replaced-by-mandatory-base-package-manual-intervention-required) it is required to install a kernel along with the base package. Else the mkinitcpio package will not be installed and you will face errors in the installation ahead.

```html
pacstrap /mnt base linux linux-firmware
```
## Mount EFI Partition
```html
mkdir -p /mnt/boot/efi
```

Check partiton table
```html
gdisk -l /dev/sda
```

<pre>
Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048          923647   450.0 MiB   2700  Basic data partition
   2          923648         1128447   100.0 MiB   EF00  EFI system partition
   3         1128448         1161215   16.0 MiB    0C01  Microsoft reserved ...
   4         1161216       133424499   63.1 GiB    0700  Basic data partition
   5       133425152       135167999   851.0 MiB   2700  
   6       135170048       344066047   99.6 GiB    0700  Basic data partition
   7       344066048       365037567   10.0 GiB    8200  swap
   8       365037568       500118158   64.4 GiB    8300  root
</pre>

EFI partiton number is 2
```html
mount /dev/sda2 /mnt/boot/efi
```

## Generate fstab
```html
genfstab -p /mnt >> /mnt/etc/fstab
```

Check the fstab
```html
cat /mnt/etc/fstab
```

## chroot into system now
```html
arch-chroot /mnt
```

## Setting Locale

The Locale defines which language the system uses, and other regional considerations such as currency denomination, numerology, and character sets. Possible values are listed in /etc/locale.gen. Uncomment en_US.UTF-8, as well as other needed localisations.

Save the file, and generate the new locales:

```html
locale-gen
echo 'LANG=en_US.UTF-8' > /etc/locale.conf
export LANG=en_US.UTF-8
```

Set the time
```html
ln -s /usr/share/zoneinfo/Asia/Kolkata /etc/localtime
```

It is recommended to adjust the time skew, and set the time standard to UTC:
```html
hwclock --systohc --utc
```

Set the hostname:
```html
echo 'arch' > /etc/hostname
```

For wifi install
```html
pacman -S iw wpa_supplicant dialog
```

Enable multilib and AUR repositories in /etc/pacman.conf:
```html
nano /etc/pacman.conf
```

Uncomment multilib (remove # from the beginning of the lines):
<pre>
[multilib]
Include = /etc/pacman.d/mirrorlist
</pre>

Add the following lines at the end of /etc/pacman.conf:
<pre>
[archlinuxfr]
SigLevel = Never
Server = http://repo.archlinux.fr/$arch
</pre>

Generate the initramfs image:
```html
mkinitcpio -p linux
```

Change root user password
```html
passwd root
```

### Then add user with your name
```html
useradd -m -g users -G wheel,storage,power,video -s /bin/bash -c "Abhishek AN" darkabhi
```

Set password for darkabhi
```html
passwd darkabhi
```

Add new user to sudoers
```html
EDITOR=nano visudo
```
Uncomment # %wheel ALL=(ALL) ALL

# GRUB INSTALLATION

I prefer GRUB 2.0 so we will be using that.

```html
pacman -Syu grub efibootmgr
```
Generate grub config
```html
grub-mkconfig -o /boot/grub/grub.cfg
```
Install grub to your drive
```html
grub-install /dev/sda
```

Verify if grub installed, type:
```html
ls -l /boot/efi/EFI/arch/
```

# POST GRUB INSTALLATION - DE/WM/Network

## Minimal GNOME DE installation
```html
pacman -S gnome-system-monitor gnome-shell nautilus gnome-terminal gnome-tweak-tool gnome-control-center xdg-user-dirs networkmanager gnome-keyring network-manager-applet xorg gdm
```

Enable GDM:
```html
systemctl enable gdm
```

Enable networking:
```html
systemctl enable NetworkManager
```

You would have noticed when we generated grub config, windows was not found. For that
Boot into arch, open terminal and as root type
```html
pacman -Syu os-prober
```

Generate grub config
```html
grub-mkconfig -o /boot/grub/grub.cfg
```
You can see it has found windows boot manager. Reboot and check now.

That's it! You have successfully dual booted arch and windows!

## Minimal KDE installation
```html
pacman -S plasma-desktop
```
