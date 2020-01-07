# karch
Kodi standalone on Arch Linux with Chrome browser and Droidmote remote control.

There is a great youtube vid on creating a new UEFI install of Arch Linux that needs a look over as opposed to older guides as its true recently some of the base packages have changed and packages once included are now found elsewhere.
You can view it here https://www.youtube.com/watch?v=-zb8220uUiA

I am going to also just add a little extra and be inclusive to total Linux noobs.
So we need to get the latest arch image https://www.archlinux.org/download/ and create either a cd/dvd or burn to a USB stick.

The current Iso at writing is http://mirrors.evowise.com/archlinux/iso/2020.01.01/archlinux-2020.01.01-x86_64.iso and either use DC/DVD burning software or grab something like Etcher or Rufus and burn to a USB stick.

Rufus is a great little utility https://rufus.ie/ and a tiny download and easy to use.
Etcher is also great often used by many raspberry and embedded users. https://www.balena.io/etcher/
Or if your on Linux such as Ubuntu use the already installed image writer.

This guide is for PC installs as for £20 -£40 there are many small format PC's available second hand that make absolutely great Kodi / Chrome machines.
Its a shame but the Kodi-Standalone-service in the AUR strangely will only work for X86_64 haven't had time to check out why it tries to create the systemd file multiple times and reurns a conflict it already exists?
I will but for now here is a guide that is currently PC only uses any android smart phone for remote control and use WoL (Wake on Lan) to wake the PC so that its easy to turn on and easy to save energy and turn off.

The HP 6300/8300 SFF machines I have used make great Kodi PC's  to plug into your widescreen TV. They have 4x USB3.0 6x USB2.0 the display adapter with a HDMI adapter also conveys audio.
Try and get an I3 as more than powerfull enough and pulls less power than I5 or higher with 4Gb ram and many exist at low prices.

Create your CD/DVD or USB stick of archlinux-xxxx.xx.xx-x86_64.iso with rufus or etcher change the bios so you boot first from DVD or USB in your boot order or press the key such as F9 to select the device to boot from just after you switch on.

 The Arch Linux Live cd will auto logon and the above video https://www.youtube.com/watch?v=-zb8220uUiA is an excellent guide where I am just going to echo the commands here and add a few things that may confuse those new to Arch Linux.

What you can do here is install openssh into the live cd so you can something like ubuntu ssh client or putty on a remote machine
```
pacman -Sy
pacman -S openssh
systemctl start sshd
ip a
```
The last command will give you the ip details to connect but you will need to give root a password
```
passwd
```
You will get a prompt for new password and to confirm it
Then connect so you can copy and paste the following.

```
timedatectl set-ntp true
```

```
fdisk -l
```

You should see something like the fllowing
```
root@archiso ~ # fdisk -l

Disk /dev/sda: 111.81 GiB, 120034123776 bytes, 234441648 sectors
Disk model: SATA SSD
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 1A91B55B-41A3-1248-BBE7-22C76A3F3465

Device       Start      End  Sectors  Size Type
/dev/sda1     2048  1050623  1048576  512M EFI System
/dev/sda3  1050624 12253183 11202560  5.4G Linux root (x86-64)




Disk /dev/sdb: 14.54 GiB, 15597568000 bytes, 30464000 sectors
Disk model: Ultra
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x00242fda

Device     Boot Start      End  Sectors  Size Id Type
/dev/sdb1  *     2048 30463999 30461952 14.5G  c W95 FAT32 (LBA)


Disk /dev/loop0: 541.5 MiB, 567787520 bytes, 1108960 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
```

/dev/sdb is the USB pen I used 16gb but the reality is they are always smaller.
/dev/sda is the 120gb SSD I am going to use as now extremely cheap and just over £15.

```
fdisk /dev/sda
```

Press g and eneter to create a new GPT disklabel.
Press n to create the first partition.
Just hit enter to accept the default of 1
Same again for the start sector hit enter
+512M and eneter will create a small first partition for the EFI partition
If you get a prompt to remove previous signatures click yes.
Press n again to create the swap partition
Just hit enter to accept the default of 2
Same again for the start sector hit enter
+8G will create an 8Gb swap for my 8Gb of memory.
Usually Swap is set to 1x-2x available memory but with Kodi it is very unlikely it will even come close to needing swap.
So you can omit swap if you wish.
Again if you get a prompt to remove previous signatures click yes.
Press n again to create the system root partition.
Just hit enter to accept the default of 3
Same again for the start sector hit enter
Also hit enter again as we will use the whole disk but generally the install will be fine in as small as 8GB.

Now we will set the partition types.
Press t and enter, 1 and enter for 1st partition, 1 and enter for EFI system.
Press t and eneter, 2 and enter for 2nd partition, 19 and enter for linux swap if used.
press t and enter 3 and eneter for 3rd partition, 24 and enter for Linux root x86_64

press w and enter and fdisk will write new partitions and exit.
You should now have something looking like this
```
130 root@archiso ~ # fdisk -l                                                                                               :(
Disk /dev/sda: 111.81 GiB, 120034123776 bytes, 234441648 sectors
Disk model: SATA SSD
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 3DF5DF4E-E40A-A947-9D33-9552BBFD37AE

Device        Start       End   Sectors   Size Type
/dev/sda1      2048   1050623   1048576   512M EFI System
/dev/sda2   1050624  17827839  16777216     8G Linux root (x86-64)
/dev/sda3  17827840 234441614 216613775 103.3G Linux filesystem
```

Format the partitions
```
mkfs.vfat /dev/sda1
mkswap /dev/sda2
swapon /dev/sda2
mkfs.ext4 /dev/sda3
```

Mount the partitions
```
mount /dev/sda3 /mnt
mkdir /mnt/boot
mount /dev/sda1 /mnt/boot
```

Now put the base system onto the root partition.
```
pacstrap /mnt base base-devel linux linux-firmware vim vi nano man-db man-pages inetutils netctl dhcpcd s-nail intel-ucode openssh
```
The above was for a Intel cpu system if AMD then change intel-ucode to amd-ucode in the above.

Create /etc/fstab
```
genfstab -U /mnt >> /mnt/etc/fstab
```

Chroot into our install
```
arch-chroot /mnt
```

Set the time zone `ls /usr/share/zoneinfo/` if you need to list available
I am in the UK so
```
ln -sf /usr/share/zoneinfo/Europe/London /etc/localtime
```

Run hwclock to generate /etc/adjtime:
```
hwclock --systohc
```

Uncomment en_GB.UTF-8 UTF-8 in /etc/locale.gen, and generate them with:
```
nano /etc/locale.gen
locale-gen
nano /etc/locale.conf
```
Paste the following
```
LANG=en_GB.UTF-8
```
Set the keyboard map
```
nano /etc/vconsole.conf
```
Paste the following
```
KEYMAP=uk
```
Create the hostname file:
```
nano /etc/hostname
```
Paste the following
```
kodi
```
Add matching entries to hosts
```
nano /etc/hosts
```
Paste the following
```
127.0.0.1	localhost
::1		localhost
127.0.1.1	kodi.localdomain	kodi
```

Set the root password (simple for example)
```
passwd
```
enter password and confirm password

Install the bootloader and configure
```
bootctl --path=/boot install
cd boot
cd loader
nano loader.conf
```
loader.conf will look something like this
```
#timeout 3
#console-mode keep
default fd2268e7ec4b44e3af6b0a86b35005e2-*
```
Change to
```
#timeout 3
#console-mode keep
default arch-*
```
Create an entry
```
cd entries
nano arch.conf
```
paste the following
```
title   Arch Linux
linux   /vmlinuz-linux
initrd  /intel-ucode.img
initrd  /initramfs-linux.img
options root=UUID=
```
type 'blkid' and eneter and copy the UUID between the quotes of the root partition
So it looks something like the following
```
title   Arch Linux
linux   /vmlinuz-linux
initrd  /intel-ucode.img
initrd  /initramfs-linux.img
options root=UUID=90a8f636-81d9-4433-af65-777f43a49be7 rw
```

Make sure the services start on boot and create temp admin user for sudo
```
systemctl enable dhcpcd.service
systemctl enable sshd
useradd -G wheel admin
```
enter password and confirm

```
visudo
```
Uncomment this line
```
%wheel      ALL=(ALL) ALL
```
So that wheel members can use sudo
```
reboot
```

Now hopefully we should have a working base system for Arch 















