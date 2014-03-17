---
layout: post
title: "Full Disk Encryption in Arch Linux with UEFI"
date:  2014-03-16
category: tech
tags: tech privacy security arch linux full disk encryption uefi dm-crypt
---

This is a guide for those of you who are interested in setting up a full disk encrypted Arch Linux environment on a UEFI capable system.  The Arch Wiki has fantastic information on all of the procedures outlined below, but I thought it would be nice to have all of this information condensed.  One thing I intentionally left out of this guide was how to securely prepare your hard drive for use before beginning.  The Arch Wiki has a very straightforward article on that [which you should read][1] before proceeding.

This first step is to get our language and locale set up.
{% highlight bash %}
nano /etc/locale.gen
	# File contents
    en_US.UTF-8 UTF-8
locale-gen
nano /etc/locale.conf
	# File contents
    LANG=en_US.UTF-8
export `cat /etc/locale.conf`
{% endhighlight %}

Next we set up partitions using gdisk.
{% highlight bash %}
gdisk /dev/sda
{% endhighlight %}

You will want to create two partitions, one with an EFI type code (ef00) and the other with the regular Linux type code (8300).  Here is what my GPT looks like.
{% highlight text %}
gdisk -l /dev/sda
Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048         1050623   512.0 MiB   EF00  EFI System
   2         1050624       250069646   118.7 GiB   8E00  Linux LVM
{% endhighlight %}

After that we will want to create and mount our LUKS container.  These commands will use the defaults for dm-crypt, but there are many more options [to choose from][2].
{% highlight bash %}
cryptsetup luksFormat /dev/sda2
cryptsetup open --type luks /dev/sda2 lvm
{% endhighlight %}

Next we create and mount our filesystems.
{% highlight bash %}
mkfs.vfat /dev/sda1
pvcreate /dev/mapper/lvm
vgcreate vol0 /dev/mapper/lvm
# Adjust swap size according to your system needs.
lvcreate --name lv_swap -L 6GB vol0
lvcreate --name lv_root -l 100%FREE vol0
mkswap /dev/mapper/vol0-lv_swap
swapon /dev/mapper/vol0-lv_swap
mkfs.ext4 /dev/mapper/vol0-lv_root
mount /dev/mapper/vol0-lv_root /mnt
mkdir /mnt/boot
mount /dev/sda1 /mnt/boot
{% endhighlight %}

Then set up the base system.  Make sure you have a working network connection before you begin this part.
{% highlight bash %}
pacstrap -i /mnt base base-devel
getnfstab -U -p /mnt >> /mnt/etc/fstab
{% endhighlight %}

At this point we have bootstrapped the environment we will boot into enough to start working within it.  Some of this will look familiar from the beginning of the setup process, but that's because we still need to configure languages within our boot environment.
{% highlight bash %}
arch-chroot /mnt /bin/bash
nano /etc/locale.gen
	# File contents
    en_US.UTF-8 UTF-8
locale-gen
nano /etc/locale.conf
	# File contents
    LANG=en_US.UTF-8
export `cat /etc/locale.conf`
{% endhighlight %}

The following commands set up a few more system variables such as system time, passwords, and hostname.  You will want to adjust them according to your preferences.
{% highlight bash %}
ln -s /usr/share/zoneinfo/US/Pacific-New /etc/localtime
hwclock --systohc --utc
passwd
echo "archie" > /etc/hostname
{% endhighlight %}

Make sure to set up the package manager to allow the multilib repository.
{% highlight bash %}
nano /etc/pacman.conf
{% endhighlight %}

Uncomment these two lines.
{% highlight text %}
[multilib]
Include = /etc/pacman.d/mirrorlist
{% endhighlight %}

The next step is to update the package lists and install the gummiboot application.  The last command sets up the initial gummiboot environment.
{% highlight bash %}
pacman -Sy
pacman -S gummiboot
gummiboot install
{% endhighlight %}

Now we will find the UUID of /dev/sda2 and insert it into the file /boot/loader/entries/arch.conf.  Then edit arch.conf to include proper kernel boot parameters.
{% highlight bash %}
blkid /dev/sda2 | awk '{print $2}' | sed 's/"//g' > /boot/loader/entries/arch.conf
nano /boot/loader/entries/arch.conf
{% endhighlight %}

Here is the full content of my arch.conf file.  The allow-discards portion enables TRIM support and is only needed if you have an SSD.  There is a security trade-off for performance when using TRIM, so please [read this][3] to decide if it's worth the trade-off for you.  The resume= option will enable hibernation on the device.  The nice thing about having an encrypted swap partition is that your hibernation data will be encrypted just like the rest of the at-rest data.  This makes hibernation a very secure alternative to leaving your machine in stand-by mode, which is vulnerable to the [cold boot attack][4].
{% highlight text %}
title		Arch Linux
linux		/vmlinuz-linux
initrd		/initramfs-linux.img
options 	cryptdevice=UUID=54cb9e06-cd82-4e90-8d21-62b4cc34f3d3:lvm:allow-discards resume=/dev/mapper/vol0-lv_swap root=/dev/mapper/vol0-lv_root rw quiet
{% endhighlight %}

Now update your your .efi boot files using the gummiboot command.
{% highlight bash %}
gummiboot update
{% endhighlight %}

You will need to modify the mkinitcpio.conf file to include several important kernel modules.
{% highlight bash %}
nano /etc/mkinitcpio.conf
{% endhighlight %}

Add keymap, encrypt, lvm2, and resume to the HOOKS paramater.
{% highlight text %}
...
HOOKS="base udev autodetect modconf block keymap encrypt lvm2 resume filesystems keyboard fsck"
...
{% endhighlight %}

Next you will need to regenerate your ramdisk.
{% highlight bash %}
mkinitcpio -p linux
{% endhighlight %}

All that is left to do now is gracefully reboot.
{% highlight bash %}
exit
umount /mnt/boot
umount /mnt
reboot
{% endhighlight %}

If you have any comments or suggestions on how to improve the instructions here, please let me know and I will consider them.  I hope this guide is as useful to you as it is to me.

[1]: https://wiki.archlinux.org/index.php/Dm-crypt/Drive_Preparation "dm-crypt/Drive Preparation"
[2]: https://wiki.archlinux.org/index.php/Dm-crypt/Device_Encryption "dm-crypt/Device Encryption"
[3]: https://wiki.archlinux.org/index.php/Dm-crypt/Specialties#Discard.2FTRIM_support_for_solid_state_drives_.28SSD.29 "Discard/TRIM support for solid state drives (SSD)"
[4]: http://en.wikipedia.org/wiki/Cold_boot_attack "Cold boot attack article on wikipedia"