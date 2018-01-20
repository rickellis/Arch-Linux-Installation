# Arch-Linux-Installation-Guide

These are the steps necessary to install <a href="https://www.archlinux.org/">Arch Linux</a> with <a href="https://en.wikipedia.org/wiki/Linux_Unified_Key_Setup">LUKS</a> disk encryption using <a href="https://en.wikipedia.org/wiki/Logical_Volume_Manager_(Linux)">Logical Volume Manager (LVM)</a> under <a href="https://en.wikipedia.org/wiki/Unified_Extensible_Firmware_Interface">EFI</a>.


### <a href="https://www.archlinux.org/download/">Download</a> the ISO and create a bootable USB
The simplest way to create a bootable USB on Linux is using the dd command:

	sudo dd bs=4M if=/path_to_arch_.iso of=/dev/sdX && sync


### Disable Secure Boot
Hold F12 during startup to access bios. If secure boot is enabled it must be turned off since Linux boot loaders don't typically have digital signatures. Secure boot is enabled by default on most modern Windows hardware. Note that if you intend on running a dual-boot system with Windows and Linux you won't be able to use disk encryption on the partition containing Windows, as it requires secure boot.


### Boot Arch Linux from the live USB
Hold F12 during startup to access startup menu. Select the USB drive and boot into Arch.


### Establish an internet connection
The most reliable way is to use a wired connection, as Arch is setup by default to connect to DHCP. To test your wired connection:
	
	ping -c 3 www.google.com

To connect to a WiFi network:

	wifi-menu


### Make sure EFI is running
Most modern systems use EFI instead of MBR to boot. The disk partitioning is slightly different on EFI systems. This document assumes EFI. If EFI is running, the following command should show output:

	efivar -l



### Zero the hard drive
First, delete any existing partitions from the drive. To see how your drive is partitioned use `fdisk -l`. If you need to remove partitions use;

	parted -s /dev/sda rm 1
	parted -s /dev/sda rm 2
	parted -s /dev/sda rm 3
	etc.

Then zero the drive with random data:

	dd if=/dev/urandom of=/dev/sd* status=progress


### Determine which drive partition you will be using

	fdisk -l


### Partition the drive




 
### Format root and home partitions using
    mkfs.ext4 /dev/sdxX

### Format/Enable SWAP
    mkswap /dev/sdaZ
    swapon /dev/sdaZ

### Check partitions with this command
lsblk /dev/sdx
 
### Mount root and home partitions
mount /dev/sdxX /mnt
mkdir /mnt/home
mount /dev/sdxY /mnt/home
 
### Choose download mirror (optional)
nano /etc/pacman.d/mirrorlist
 
### Install base packages
pacstrap -i /mnt base base-devel
 
### Configure fstab (run only once!) and verify it
genfstab -U -p /mnt >> /mnt/etc/fstab
nano /mnt/etc/fstab
 
### Change to your root directory
arch-chroot /mnt
 
### Language and Time Zone settings (find and un-comment)
nano /etc/locale.gen
locale-gen
echo LANG=en_US.UTF-8 > /etc/locale.conf
export LANG=en_US.UTF-8
 s
ls /usr/share/zoneinfo/
ln -s /usr/share/zoneinfo/<zone>/ /etc/localtime
hwclock --systohc --utc
 
### Set hostname
Note: Change "arch" to whatever you want your host to be.
	echo arch > /etc/hostname
 
### Create users (create admin psswd)
### useradd -m -g [initial_group] -G [additional_groups] -s [login_shell] [username]
useradd -m -g users -G wheel,storage,power -s /bin/bash 
passwd <username>
passwd
 
### Install sudo / add user to the admin-group
pacman -S sudo bash-completion
EDITOR=nano visudo
### and un-comment: 
%wheel ALL=(ALL) ALL
 
 
### Enable multilib repositories: find and un-comment both lines
nano /etc/pacman.conf
[multilib]
Include = /etc/pacman.d/mirrorlist
### Add Yaourt repository (same file pacman.conf)
[archlinuxfr]
SigLevel = Never
Server = http://repo.archlinux.fr/$arch
### and install yaourt
pacman -Sy yaourt customizepkg rsync
 
### Advanced Linux Sound Architecture (ALSA)
pacman -S alsa-utils pulseaudio alsa-oss alsa-lib
amixer sset Master unmute
### to test sound
speaker-test -c 2
 
### Install Boot-loader (grub)
pacman -S grub
grub-install --target=i386-pc --recheck /dev/sdx
pacman -S os-prober
grub-mkconfig -o /boot/grub/grub.cfg
 
### Enable ethernet interface service at boot (find interface with ip link)
systemctl enable dhcpcd@.service