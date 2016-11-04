# Multiboot USB

What is a [multiboot] (https://en.wikipedia.org/wiki/Multi-booting) USB? It's a USB with more than one distribution.

There are plenty of tools to create a multiboot USB. 
* [Multiboot] (http://liveusb.info/): It works well on Debian and Debian based distros. You can add a windows ISO.
* [YUMI] (http://www.pendrivelinux.com/yumi-multiboot-usb-creator/): It works on Windows.

![openSUSE USB](/pictures/opensuse_usb.jpg)

The best way to create a multiboot USB is by terminal. Everytime there is new version of the distro, all you have to do is to change the ISO file using copy>paste and change the name in the grub.cfg file.
This tutorial is based on [Arch wiki] (https://wiki.archlinux.org/index.php/Multiboot_USB_drive).

1. Create a FAT32 partition. I choose ext4 because I want to use DVDs.

Find where the usb is mounted.
```
$ cat /proc/partitions
```

or
```
$ lsblk
```

Unmount your USB.
```
$ sudo umount /dev/sdX1
```

For FAT32 use the command
```
$ sudo mkfs.vfat -n MULTIBOOT /dev/sdX1
```

For ext4 use the command (be careful. When you copy the iso to the folder, you need to have to be administrator)
```
$ sudo mkfs.ext4 -L MULTIBOOT /dev/sdX1
```

2. Mount the USB and create 2 folders, boot and iso. The first will have the nessesary files to boot and the second the ISO files of the distributions.

For Debian/Ubuntu
```
$ sudo mkdir /media/USERNAME/MULTIBOOT/{boot,iso}
```

For Arch Linud/openSUSE
```
$ sudo mkdir /run/media/USERNAME/MULTIBOOT/{boot,iso}
```

Where USERNAME is the username you have to enter your system. If the above are confusing, just go to your USB and add the two directories using nautilus (if it's ext4 as superuser).


3. Install grub.

For Debian/Ubuntu
```
$ sudo grub-install --force --no-floppy --root-directory=/media/USERNAME/MULTIBOOT/boot /dev/sdX
```

For Arch Linux
```
$ sudo grub-install --target=i386-pc --recheck --boot-directory=/run/media/USERNAME/MULTIBOOT/boot /dev/sdX
```

For openSUSE
```
$ sudo grub2-install --target=i386-pc --recheck --boot-directory=/run/media/USERNAME/MULTIBOOT/boot /dev/sdX
```

Where USERNAME is the username you have to enter your system.


4. Now copy the ISOs inside the iso folder. 

5. You have to create the grub.cfg (/boot/grub). You can copy mine from the github.
You can add # to each menuentry you don't need.

For Debian/Ubuntu
```
$ sudo nano /media/USERNAME/MULTIBOOT/boot/grub/grub.cfg
```

For Arch Linux
```
$ sudo nano /run/media/USERNAME/MULTIBOOT/boot/grub/grub.cfg
```

For openSUSE
```
$ sudo nano /run/media/USERNAME/MULTIBOOT/boot/grub2/grub.cfg
```

Where USERNAME is the username you have to enter your system.

For openSUSE Add the following text

```
# Config for GNU GRand Unified Bootloader (GRUB)
# /boot/grub/grub.cfg

# Timeout for menu
set timeout=30

# Default boot entry
set default=0

# Menu Colours
set menu_color_normal=white/black
set menu_color_highlight=white/green

# Path to the partition holding ISO images (using UUID)
#set imgdevpath="/dev/disk/by-uuid/UUID_value"
# ... or...
# Path to the partition holding ISO images (using device labels)
#set imgdevpath="/dev/disk/by-label/label_value"
set imgdevpath="/dev/disk/by-label/MULTIBOOT"

# Boot ISOs
menuentry '[NET-DVD]openSUSE-13.1-DVD-x86_64' {
set isofile='/iso/openSUSE-13.1-DVD-x86_64.iso'
loopback loop $isofile
linux (loop)/boot/x86_64/loader/linux install=hd:$isofile
initrd (loop)/boot/x86_64/loader/initrd
}

menuentry '[LIVE]openSUSE-13.1-KDE-Live-x86_64' {
set isofile='/iso/openSUSE-13.1-KDE-Live-x86_64.iso'
loopback loop $isofile
linux (loop)/boot/x86_64/loader/linux isofrom_device=$imgdevpath isofrom_system=$isofile LANG=en_US.UTF-8
initrd (loop)/boot/x86_64/loader/initrd
}
```

If you want more, you can check the [Arch Wiki] (https://wiki.archlinux.org/index.php/Multiboot_USB_drive) or use my grub.cfg example. Just put # for the menuentries you don't need.

I will describe 2 examples here that need attention.

1. Debian: Download **firmware-8.6.0-amd64-i386-netinst.iso** and put it in the iso folder.

I had to download [initrd.gz for 64bit] (https://mirrors.kernel.org/debian/dists/stable/main/installer-amd64/current/images/hd-media/initrd.gz) and 
[initrd.gz for 32bit] (https://mirrors.kernel.org/debian/dists/stable/main/installer-i386/current/images/hd-media/initrd.gz)

Rename them accordingly and copy them to **iso** folder.<br>
As menuentry I used for 64bit
```
menuentry "Debian 8.6 64bit" {
	set isofile='/iso/firmware-8.6.0-amd64-i386-netinst.iso'
	set initrdfile='/iso/firmware-8.6.0-amd64-netinst.initrd.gz'
	loopback loop $isofile
	linux (loop)/install.amd/vmlinuz iso-scan/ask_second_pass=true iso-scan/filename=$isofile vga=788 -- quiet
	initrd $initrdfile
}
```

and
```
menuentry "Debian 8.6 32bit" {
	set isofile='/iso/firmware-8.6.0-amd64-i386-netinst.iso'
	set initrdfile='/iso/firmware-8.6.0-i386-netinst.initrd.gz'
	loopback loop $isofile
	linux (loop)/install.386/vmlinuz iso-scan/ask_second_pass=true iso-scan/filename=$isofile vga=788 -- quiet
	initrd $initrdfile
}
```

I pointed where the **initrd.gz** file is.

 [Hirens Boot CD] (http://www.hirensbootcd.org/): This is complicated.

- Download the cd
```
wget http://www.hirensbootcd.org/files/Hirens.BootCD.15.2.zip 
```

- Unzip the file **Hirens.BootCD.15.2.zip**

- Open the ISO with the unzip program and unzip the folder **HBCD**.

![Unzip HBCD](/pictures/1.jpg)

- Copy the folder **HBCD** to the **root** of your **USB**. Be careful, your **USB** has to be **FAT32 or NTFS** to work. The openSUSE ISOs don't work with NTFS. So you have to decide what to use.


- Fo to the folder **HBCD/Dos/** and decompress the file **dos.gz**. You'll have the file **dos.img**

![Unzip dos.gz](/pictures/2.jpg)

- Mount the file **dos.img** (use **Disk Image Mounter**) and copy the file **grub.exe** to your **root** of the **USB**.

![Mount dos.img](/pictures/3.jpg)

- Copy the file **menu.lst** to the root of the USB (optional).

- Your USB has the following folders and files:
```
boot
HBCD
iso
menu.lst
grub.exe 
```

![USB contents](/pictures/4.jpg)

- Finally add the menuentry to **grub.cfg**.
```
menuentry "HIRENS TOOLS" { 
linux16 /grub.exe —config-file="find —set-root /HBCD/menu.lst; configfile /HBCD/menu.lst" 
} 
```

More info
* [Arch wiki](https://wiki.archlinux.org/index.php/Multiboot_USB_drive)
* [vancepym/MULTIBOOT](https://github.com/vancepym/MULTIBOOT/tree/master/grub2)

Here are the tutorials in Greek:
* [Δημιουργία multi liveUSB με openSUSE] (http://eiosifidis.blogspot.gr/2015/05/script-multi-liveusb-opensuse.html)
* [Δημιουργία Multiboot USB, συνέχεια--> Προσθήκη Hirens] (http://eiosifidis.blogspot.gr/2016/10/multiboot-usb-hirens.html)
