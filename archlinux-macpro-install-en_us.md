![alt text][logo]

# Installation on Apple Mac Pro 5,1



- [Introduction](##introduction)
- [Pre-installation](##pre-installation)
- [Installation](##installation)



## Introduction



This guide walks you through the process of installing Arch Linux 2021.04.01 on an Apple Mac Pro with the following specifications: 

- Model: A1289
- Family: Mid-2010
- ID: MacPro5,1

- Processors: 2 Xeon X5650 2.66 GHz 6 Core
- ROM/Firmware Type: EFI 64-Bit
- RAM: 32 GB 1333 MHz DDR3 ECC SDRAM
- Storage: 1 TB SSD / 3 TB HDD
- Video Card: AMD Radeon RX 580 8 GB VRAM
- Standard Ethernet: 2 Gigabit 10/100/1000BASE-T
- Wireless: AirPort Extreme (802.11a/b/g/n) and Bluetooth 2.1+EDR is standard
- Optical: 18x DL "Superdrive"



## Pre-installation



### Acquire an installation image and verify signature

Visit the [Download](https://archlinux.org/download/) page and, depending on how you want to boot, acquire the ISO file or a netboot image, and the respective [GnuPG](https://wiki.archlinux.org/index.php/GnuPG) signature. Then [verify](https://wiki.archlinux.org/index.php/GnuPG#Verify_a_signature) it with:

```
$ gpg --keyserver-options auto-key-retrieve --verify archlinux-version-x86_64.iso.sig
```

Alternatively, from an existing Arch Linux installation run:

```
$ pacman-key -v archlinux-version-x86_64.iso.sig
```

### Prepare an installation medium

The installation image can be supplied to the target machine via a [USB flash drive](https://wiki.archlinux.org/index.php/USB_flash_installation_medium), an [optical disc](https://wiki.archlinux.org/index.php/Optical_disc_drive#Burning) or a network with [PXE](https://wiki.archlinux.org/index.php/PXE): follow the appropriate article to prepare yourself an installation medium from the chosen image.

### Boot the live environment

1. Ethernet—plug in the cable.

2. Point the current boot device to the one which has the Arch Linux installation medium. Typically it is achieved by pressing a key during the POST phase, as indicated on the splash screen. Refer to your motherboard's manual for details.

3. When the installation medium's boot loader menu appears, select Arch Linux install medium and press Enter to enter the installation environment.

4. You will be logged in on the first [virtual console](https://en.wikipedia.org/wiki/Virtual_console) as the root user, and presented with a [Zsh](https://wiki.archlinux.org/index.php/Zsh) shell prompt.

5. Verify the connection with [ping](https://en.wikipedia.org/wiki/ping_(networking_utility)):

   ```
   # ping archlinux.org
   ```

> **Note**: Arch Linux installation images do not support Secure Boot. You will need to disable Secure Boot to boot the installation medium. If desired, Secure Boot can be set up after completing the installation.

### Setting up the environment 

#### Set the keyboard layout

```
# loadkeys us-acentos
```

#### Set the default editor

```
# export EDITOR=nano
```


#### Update the system clock

```
# timedatectl set-ntp true
```

### Drive Preparation

#### Partition the drive

When recognized by the live system, disks are assigned to a block device, such as /dev /sda or /dev /nvme0n1. To identify these devices, use fdisk. 

```
# fdisk -l
```

Following is my disk set-up. I will show you how to create a swap file later. Here the disk I will use will be /dev/sda. 

Use cfdisk to partition the disk using the following layout: 

```
# /dev/sda
```

| Size | Partition | Partition Code       | Partition Code      | File System |
| ---- | --------- | -------------------- | ------------------- | ----------- |
| 512M | /dev/sda1 | EFI system partition | UEFI Boot Partition | FAT32       |
| *    | /dev/sda2 | Linux File System    | Linux File System   | xfs         |

> (*) use the remaining disk space.

#### Formatting the drives

```
# mkfs.vfat -F32 /dev/sda1
# mkfs.xfs -f /dev/sda2 
```

#### Mounting drives for install

```
# mount /dev/sda2 /mnt
# mkdir /mnt/boot
# mount /dev/sda1 /mnt/boot
# lsblk 
```

#### Find the best mirror list for downloading Arch Linux

```
# pacman -Sy --noconfirm reflector
# reflector --verbose --latest 50 --protocol https --threads 24 --sort rate --save /tmp/mirrorlist.new
# rankmirrors -n 0 /tmp/mirrorlist.new > /tmp/mirrorlist && sudo cp /tmp/mirrorlist /etc/pacman.d/
# cat /etc/pacman.d/mirrorlist
sudo pacman -Syu
```



## Installation



### Essential packages

Use the [pacstrap](https://man.archlinux.org/man/pacstrap.8) script to install the base package, Linux kernel and firmware for common hardware:

```
# pacstrap /mnt base base-devel linux linux-firmware linux-headers intel-ucode dosfstools xfsprogs sudo git
```

#### Fstab

Generating fstab for the drives:

```
# genfstab -U -p /mnt >> /mnt/etc/fstab
```

Creating RAM Disk:

```
echo "tmpfs	/tmp	tmpfs	rw,nodev,nosuid,size=2G	0 0" >> /mnt/etc/fstab
```

Creating the swap file:

```
# dd if=/dev/zero of=/mnt/swapfile bs=1M count=34816 status=progress
# chmod 600 /mnt/swapfile
# echo "/swapfile    none    swap    defaults    0 0" >> /mnt/etc/fstab
```

#### Enter the new system

```
# arch-chroot /mnt
```

### Configure the system

#### Setting machine name

```
# echo Discovery > /etc/hostname
# echo "127.0.0.1 localhost.localdomain	localhost" >> /etc/hosts
# echo "::1       localhost.localdomain	localhost" >> /etc/hosts
# echo "127.0.1.1 Discovery.localdomain	Discovery" >> /etc/hosts
```

#### Setting font for vconsole

```
# echo "KEYMAP=us-acentos" >> /etc/vconsole.conf
```

#### Setting system wide language

```
# echo LANG=en_US.UTF-8 >> /etc/locale.conf
# echo LC_COLLATE=C >> /etc/locale.conf
# sed -i '/#en_US.UTF-8/s/^#//g' /etc/locale.gen
# locale-gen
```

#### Time zone

```
# ls -sf /usr/share/zoneinfo/America/Sao_Paulo /etc/localtime
# hwclock --systohc --utc
```

#### Setting modules

```
# echo coretemp > /etc/modules
# echo applesmc >> /etc/modules
```

#### Setting environment variables

```
# echo EDITOR=nano visudo >> /etc/environment
# echo GTK_IM_MODULE=cedilla >> /etc/environment
# echo QT_IM_MODULE=cedilla >> /etc/environment
```

#### Set root password

```
# passwd
```

#### Create a user

```
# useradd  -m -g users -G wheel,sys,log,network,floppy,scanner,power,rfkill,users,video,storage,optical,lp,audio,adm,ftp,mail,git -s /bin/zsh MYUSERNAME
# passwd MYUSERNAME
```

#### Giving user wheel access

```
# sed -i '/%wheel ALL=(ALL) NOPASSWD: ALL'/s/^#//g /etc/sudoers
```

#### Installing the bootloader

```
# pacstrap grub
```











[logo]: https://archlinux.org/static/logos/archlinux-logo-black-90dpi.0c696e9c0d84.png "Arch Linux"

