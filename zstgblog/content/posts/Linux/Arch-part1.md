+++
title = "Installing Arch the fun way"
author = "ZeStig"
authorTwitter = "" #do not include @
cover = ""
tags = ["Arch", "Linux"]
keywords = ["Linux", "Arch"]
description = "In this blog post, we check out the installation of Archlinux, a distro notoriously hard to install."
showFullContent = false
readingTime = false
hideComments = false
color = "" #color from the theme settings
+++

Installing Arch is not hard at all, it's quite simple to do. Arch has long been known as a power-users' distro, and Arch users are usually elitists. Today, we'll break that notion! 

Let's learn how to install Arch manually!

## Step 1: Download the Arch ISO

Download the Arch ISO from [here](https://archlinux.org/download). You can also download it from a mirror in your country for faster download speeds.

## Step 2: Boot into the Arch ISO

Create a fresh VM and boot into the Arch ISO that you've downloaded. Alternatively, you can flash the image to a USB and boot from it. The ISO file will automatically boot into a shell. You can type `clear` to clear the screen.

## Step 3: Set the Time and Date

Run the following command to set the time and date:

```bash
timedatectl set-ntp true
```

## Step 4: Partition the Virtual Hard Drive

Partition your virtual hard drive using the following commands:

```bash
parted /dev/sda -- mklabel gpt # THIS WILL WIPE YOUR DEVICE, don't do this if you're dual-booting
parted /dev/sda -- mkpart ESP fat32 1MB 512MB # 512 MB bootloader
parted /dev/sda -- mkpart primary ext4 512MB 100% # rem space for the root filesystem
parted /dev/sda -- set 1 esp on # make the partition bootable
```

## Step 5: Format the Partitions

Format the partitions that you've created:

```bash
mkfs.fat -F32 /dev/sda1 -n boot
mkfs.ext4 /dev/sda2 -L root
```

If you've created a btrfs filesystem, use the following command instead:

```bash
mkfs.btrfs /dev/sda2 -L root
```

## Step 6: Mount the Partitions

Mount the partitions to your filesystem:

```bash
mount /dev/sda2 /mnt
mkdir -p /mnt/boot/EFI
mount /dev/sda1 /mnt/boot/EFI
```

## Step 7: Install Base Arch Stuff

Install the base Arch stuff onto your install. This process can take ≈ 10-15 min depending on your internet connection:

```bash
pacstrap /mnt base linux linux-firmware
```

## Step 8: Set Hostname

Add a hostname to your VM:

```bash
genfstab -L /mnt >> /mnt/etc/fstab # use labels to identify em
modprobe efivarfs
arch-chroot /mnt
```

Choose a proper timezone and set the hostname:

```bash
ln -sf /usr/share/zoneinfo/America/Chicago /etc/localtime
hwclock --systohc
echo Arch-VM > /etc/hostname # replace with a hostname of your choice, preferably no spaces
```

## Step 9: Configure /etc/hosts

Edit the `/etc/hosts` file and add the following:

```bash
127.0.0.1   localhost
::1         localhost
127.0.1.1   arch.localdomain arch # REPLACE arch WITH YOUR HOSTNAME!!!
```

## Step 10: Set Root Password

Set the password for the root user:

```bash
passwd
```

## Step 11: Add User

Add a user and set a password for it:

```bash
useradd -m stig && passwd stig:
```

Add the user to the necessary groups:

```bash
usermod -aG wheel,audio,video,optical,storage stig
```

Install sudo and nano:

```bash
pacman -S sudo nano --noconfirm
```

Add the user to the wheel group in the `/etc/sudoers` file:

```bash
EDITOR=nano visudo
```

Uncomment the following line in the `/etc/sudoers` file:

```bash
%wheel ALL=(ALL:ALL) ALL
```

## Step 12: Install Bootloader

|      | Systemd-boot   | GRUB |
| --   |  --            | --   |
| BIOS | N              | Y    |
| UEFI | Y              | Y    | 

Install the bootloader:

```bash
pacman -S grub efibootmgr os-prober
```

Note that os-prober is optional, but can be useful to troubleshoot dual-boot systems. You need not install it inside a VM, or when you're not dual-booting.
Then install the bootloader to the right place:
```bash
mkdir /mnt/boot/EFI 
grub-install --target=x86_64-efi --bootloader-id=grub_uefi --recheck 
```

Now we need to generate a grub config file.
```bash
grub-mkconfig -o /boot/grub/grub.cfg
```
Get a networking daemon set up:
`pacman -S networkmanager --noc && systemctl enable NetworkManager`

Leave the arch-chroot; installation's almost over.

`exit`

Finally unmount the live USB: `umount -l /mnt && exit`

Shut down the VM: `shutdown now`

The virtual machine has been shut down , eject the ISO file (remove the USB if you're performing the installation on bare metal).
When the VM has finished booting up once more, login into the tty by typing your username and password.

Next, we're going to install video drivers (*if* they haven't been installed):

`sudo pacman -S --needed xf86-video-fbdev xorg xorg-xinit --noc`

Now we install a desktop.

Choose which DE you need from [this link](https://wiki.archlinux.org/title/Desktop_environment).

<!--
```
cp /etc/X11/xinit/xinitrc $HOME/.xinitrc
echo "exec /usr/bin/<DE>-session" >> ~/.xinitrc
sudo reboot
startx # to start the display server
```
-->
 If you want a [display manager](https://wiki.archlinux.org/title/Display_manager) such as [SDDM](https://github.com/sddm/sddm), you can set it up as well:
```
sudo pacman -S sddm --noconfirm
sudo systemctl enable sddm # will autostart the service from next boot onwards
```
Now that the Arch install **has _finished_**, simply reboot your system using the `reboot` command.