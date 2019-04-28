---
layout: post
title:  "Installing Arch Linux on VirtualBox (With networking)"
date:   April 28, 2019
categories: project tutorial
excerpt_separator: <!--more-->
---
A couple of weeks ago I decided to try to install Arch Linux as a virtual machine in [VirtualBox](http://virtualbox.org). I am not a pro with Linux, so I knew that getting Arch Linux to work was going to be a challenge. Installing wasn’t terribly difficult; it only took me one day to figure out how to manage partitions with `fdisk`. The hard part was getting Arch Linux connected to the internet (and not just the local network). After tinkering around with it for about a day, I finally out what was preventing the virtual machine from connecting to the internet. I created this guide for people who want to mess around with Arch Linux, but don’t have time (or are too lazy) to figure out how to install Arch Linux from their [wiki](https://wiki.archlinux.org). Don’t worry, I have shortened the instructions so that they don’t include all of the trial-and-error to get the internet working. Note that this tutorial was created for macOS hosts, but can be easily adapted for Windows and Linux hosts.<!--more--> ::write something about the non gui arch and BIOS::

## Create A New VM for Arch Linux
The first step in getting Arch Linux running, is to create a virtual computer that you can run Arch Linux from. Make sure that you have [Oracle VirtualBox](https://virtualbox.org) installed on your computer first. The next thing you have to do is adjust a couple of the settings for the virtual machine to make it more useable once installed.

### Create a New VirtualBox Virtual Machine:
1. Click **New** to create a new virtual machine
![Naming the VM](/assets/posts/2019/04/28/installing-arch-linux-in-virtualbox-with-networking/Naming.png)
2. Click **Expert Mode**, this makes for fewer steps in the setup process
3. Name the virtual machine something memorable. (**Tip:** If you name the VM something with “arch” in the name, VirtualBox will automatically set some of the default options)
4. Change the **Type** to Linux
5. Change the **Version** to Arch (64bit) (or 32bit if that is the version you downloaded)
6. Give the VM at least 2GB of memory (2048MB)
![Adjusting the amount of RAM for the VM](/assets/posts/2019/04/28/installing-arch-linux-in-virtualbox-with-networking/Setting RAM.png)
7. Click **Create**
8. Change the disk size to something above 16GB (this gives you plenty of room to install programs and create files)
![Setting the size of the hard drive for the vm](/assets/posts/2019/04/28/installing-arch-linux-in-virtualbox-with-networking/Setting HDD.png)
9. Click **Create**

### VirtualBox Settings To Change
Click **Settings** to adjust aspects of the selected virtual machine.

1. In the **System** tab under _Motherboard_, make sure the **Base Memory** is set to 4GB (4096), **Enable I/O APIC** is enabled, and **Enable EFI** is disabled
2. Also in the **System** tab under _Processor_, make sure that the number of CPU cores is set to 4 (or 2 if you have a computer with less power)
3. In the **Display** tab give **Video Memory** a bit more than what it is currently at (I set my VM to 128MB)
4. In the **Storage** tab, under _Controller: IDE_ select the item labeled “Empty” with the disk icon and then chose the disk icon next to the drop down and “Chose Optical Disk Image File…” Locate the Arch Linux ISO and click **Open**

![Enabling the live CD option](/assets/posts/2019/04/28/installing-arch-linux-in-virtualbox-with-networking/Live CD Enable.png)
![Menu to choose what CD file to use](/assets/posts/2019/04/28/installing-arch-linux-in-virtualbox-with-networking/Choose Live CD.png)
5. In the **Network** tab make sure _Adapter 1_ is set to **NAT**
6. In the **Ports** tab make sure **USB 3.0 (xHCI)** is selected (**Note:** This requires the VirtualBox extension pack)

## Installation
The installation process is the most difficult and time consuming part of getting Arch Linux working.

1. Start up the Arch Linux virtual machine, it should automatically boot from the Arch Linux ISO file you gave it when setting up the virtual machine and should look like the screenshot below.
![Arch Linux Live ISO boot screen](/assets/posts/2019/04/28/installing-arch-linux-in-virtualbox-with-networking/Live CD Boot.png)
2. Hit `return` to boot the Arch Linux live environment.
3. At the prompt (`root@archiso ~ #`) type `ip link` to connect to the internet (you can double check this by running `ping archlinux.org -c 5`).
4. Sync the clock with internet by running `timedatectl set-nap true`
5. Create 2 partitions using _fdisk_
	1. List the disks with `fdisk -l` (you only need to pay attention to devices ending in **nvme** or **sd_** in this tutorial I will be using the disk “sda”)
	2. Select the drive you want to partition by running `fdisk /dev/sda`
	3. Create a _swap_ partition. The _swap_ partition is used like extra RAM and allows the system to hibernate (basically a shutdown without having to do a reboot)
		1. Type `n` to create a new partition
		2. Hit `return` to set the partition to _primary_
		3. Hit `return` to set partition number to _1_
		4. Hit `return` to set the starting sector to _2048_
		5. Type `+512M` to set the ending sector to 512MB in from the start of the drive (It is recommended to set the _swap_ partition to no less than 512MB)
	4. Create the partition that the system will be installed on
		1. Type `n` to create a new partition
		2. Hit `return` to set the partition to _primary_
		3. Hit `return` to set partition number to _2_
		4. Hit `return` to set the starting sector to where the _swap_ partition ends
		5. Hit `return` to set the ending sector to the end of the drive
	5. Save and exit by typing `w`
	6. Set up and enable the _swap_ partition
		1. Type `mkswap /dev/sda1` to set up the _swap_ partition
		2. Type `swapon /dev/sda1` to enable the _swap_ partition
	7. Format the system partition with `mkfs.ext4 /dev/sda2`
	8. Mount the system partition at `/mnt` with `mount /dev/sda /mnt`
	9. Install the Arch Linux base packages to the system partition with `pacstrap /mnt base` (This can take anywhere from 5 to 15 minutes depending on your internet speed)
	10. Generate the file system tab (auto mounts disks on boot) with `genfstab -U /mnt/ >> /mnt/etc/fstab`
	11. Change root to the new system with `arch-chroot /mnt`
	12. Set the timezone that the system will use
		1. Link the timezone to the local time with `ln -sf /usr/share/zoneinfo/REAGON/CITY /etc/localtime`. Where `REAGION` and `CITY` apply to your situation. **Example**: `/usr/share/zoneinfo/America/New_York` if you live in New York time (eastern time zone)
		2. Set the hardware clock with `hwclock --systolic`
	13. Generate locale with `locale-gen`
		1. Create a new file in `/etc/` called `locale.conf` by running `nano /etc/locale.conf`
		2. Add the line `LANG=en_US.UTF-8`
		3. Exit nano with `CTRL+x` then `y` to save
	14. Give your computer a hostname
		1. Create a new file in `/etc/` called `hostname` by running `nano /etc/hostname`
		2. Type whatever you want your computer hostname to be. (**Example**: `archlinuxvm`)
		3. Exit nano with `CTRL+x` then `y` to save
		4. Edit `/etc/hosts` in nano with  `nano /etc/hosts`
		5. Add `127.0.0.1    localhost`, `::1        localhost`, and `127.0.1.1    HOSTNAME.localdomain HOSTNAME`. Where `HOSTNAME` is the hostname defined in `/etc/hostname`
	15. Prepare the boot environment with `mkinitcpio -p linux`
	16. Change the root password with `passwd` **WARNING**: If you forget your password, there is nothing you can do.
	17. Enable the _enp0s3_ service to automatically enable networking at boot with `systemctl enable dhcpcd@enp0s3.service`
	18. Install and set up the GRUB bootloader
		1. Install the GRUB installer with `pacman -S grub`
		2. Install GRUB with `grub-install --target=i386-pc /dev/sda`
		3. Generate the configuration file with `grub-mkconfig -o /boot/grub/grub.cfg`
	19. Exit _chroot_ with `exit`
	20. Remove the Live CD by going to `Devices/Optical Drives/Remove disk from virtual drive` in the menu
	21. Reboot into your new installation of Arch Linux with `reboot`
![Arch Linux GRUB bootloader screen](/assets/posts/2019/04/28/installing-arch-linux-in-virtualbox-with-networking/GRUB.png)
