# mac-arch
> Arch installation guide on Mac

# Contents

  - [Install arch dual boot](install-arch-dual-boot)
  - [Install to make arch usable](install-to-make-arch-usable)
  - [Improvement](improvement)
  - [Useful packages](useful-packages)

# Install arch dual boot

## 1. Make arch installer USB (bootable)

Download arch's iso [here](https://www.archlinux.org/download/)

Find usb by using `diskutil list`, then:

```
# Assume usb disk is /dev/diskX
diskutil unmountDisk /dev/diskX
dd if=path/to/arch.iso of=/dev/diskX bs==1m
```

## 2. Make space for Arch

Use Disk Utility Partition feature to add new Partition for Arch, follow [this guide](https://wiki.archlinux.org/index.php/Mac#Arch_Linux_with_OS_X_or_other_operating_systems)

Or if you already know how to use Disk Utility, then create a partition with FAT32 format.

## 3. Boot it up

Hold bold `alt/option` when system bootup, then choose boot from USB

:warning: If you are using Retina Macbook, tty font will be very small. To get larger font, [connect to wifi](connect-wifi) and run these commands:

```
sudo pacman -Sy terminus-font
setfont /usr/share/kbd/consolefonts/ter-132b.psf.gz
```

## 4. Connect wifi

Use `wifi-menu` then choose wifi to connect, then check connection with:

```
ping -c 3 google.com
```

## 5. Partitioning

View all your patitions to choose correct one

```
use fdisk -list
```

Open cgdisk with:

```
cgdisk /dev/sdaX
```

Create these new partitions:

  - 128MB with type Apple HFS+ (This is required in order to make thing works)
  - 256MB with type Linux filesystem
  - xMB      with type Linux Swap (If you have space, try to keep it double size of your ram size)
  - xGB       with type Linux filesystem (This is our arch place)

## 6. Format and mount partition

Assuming you have this when run `fdisk -l`:

Device             Size            Type
...                ...             ...
/dev/sda3          128MB           Apple HFS+
/dev/sda4          256MB           Linux filesystem
/dev/sda5          16GB            Linux Swap
/dev/sda6          64GB            Linux filesystem

Now let format and mount partition:

```
mkfs.ext4 /dev/sda4
mkswap /dev/sda5
mkfs.ext4 /dev/sda6
mount /dev/sda6 /mnt
mkdir /mnt/boot && mount /dev/sda4 /mnt/boot
swapon /dev/sda5
```

## 7. Install base packages & generate fstab

Run these commands:

```
pacstrap /mnt base base-devel
genfstab -p /mnt >> /mnt/etc/fstab
```

:warning: If you are using SSD drive. Open fstab config file:

```
vi /mnt/etc/fstab
```

And make sure it look like:

/dev/sda4      /boot      ext2  defaults,relatime,stripe=4           0 2
/dev/sda6      /          ext4   defaults,noatime,data=writeback     0 1

If you are using ssd, remove all discard in all lines

## 8. System config

```
arch-chroot /mnt /bin/bash
echo arch > /etc/hostname   (Change arch with your hostname)
ln -s /usr/share/zoneinfo/Asia/Ho_Chi_Minh /etc/localtime  (Use tab to select your zoneinfo easier)
useradd -m -G wheel -s /bin/bash your_username
passwd your_username   (Create your password)
```

Open /etc/sudoers file (with sudo) and uncomment this line to get sudo right for our user:
```
%wheel ALL=(ALL) ALL
```

Uncomment `en_US.UTF-8 UTF-8` (Or what ever locale you want) line in `/etc/locale.gen` file, then run:

```
locale-gen
echo LANG=en_US.UTF-8 > /etc/locale.conf
export LANG=en_US.UTF-8
```

Modify your `/etc/mkinitcpio.conf` file to insert `keyboard` after `autodetect` in the HOOK section (If it not exist). Then run:

```
mkinitcpio -p linux
```

## 8. Install the bootloader

We will boot using OSX native EFI boot loader, so install this:

```
pacman -S grub-efi-x86_64
```

Change `/etc/default/grub` to look like:

```
GRUB_CMDLINE_LINUX_DEFAULT="quiet rootflags=data=writeback"
```

Then create boot.efi with GRUB

```
grub-mkconfig -o boot/grub/grub.cfg
grub-mkstandalone -o boot.efi -d usr/lib/grub/x86_64-efi -O x86_64-efi --compress xz boot/grub/grub.cfg
```

:heavy_exclamation_mark: Important: Copy boot.efi to your usb or save it somewhere, we'll need this to duo boot.

To copy it to usb, use:

```
mkdir /mnt/myusb && mount /dev/sdb /mnt/myusb 
cp boot.efi /mnt/myusb/
```

Or upload it to file.io:

```
curl -F "file=boot.efi" https://file.io
```

:heavy_exclamation_mark: Important: If you have only wifi, please install those below, if not you'll not able to use `wifi-menu` after reboot

```
pacman -S iw wireless_tools wpa_supplicant dialog
```

Exit chroot and reboot (back to mac world)

```
exit
reboot
```

## 9. Make arch duo bootable

When OSX loaded. Using Disk Utility to format `/dev/sda3` (128MB HFS+ we have created before) with Journaled format.

Then create this file structure:
|___mach_kernel
|___System
       |___Library
              |___CoreServices
                      |___SystemVersion.plist
                      |___boot.efi              (Is the file we'v copy, upload in the previous step)

Edit SystemVersion.plist content:

```xml
<xml version="1.0" encoding="utf-8"?>
<plist version="1.0">
<dict>
    <key>ProductBuildVersion</key>
    <string></string>
    <key>ProductName</key>
    <string>Linux</string>
    <key>ProductVersion</key>
    <string>Arch Linux</string>
</dict>
</plist>
```

To make arch auto boot with out holding `alt/option`, run this command:

```
sudo bless --device /dev/disk0s4 --setBoot
```

Now when reboot and welcome to arch world :hearts:

## Install to make arch usable

## Improvement

## Useful packages
