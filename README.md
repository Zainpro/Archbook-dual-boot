
# Installing Arch Linux on MacBook Air (13-inch, Mid 2013)

The following is my dual OSX - Arch linux set up:

## Configuration PHASES

#### 1. Boot into arch and make the harddisk ready, root with password. 1st reboot
#### 2. Boot into arch, login as root and make new user. 2nd reboot
#### 3. Boot into arch,login as user and install your desktop environment.

### Create USBs for installation

#### Make a bootable USB with arch Installallation
#### Make a bootable USB with GParted (optional))

## PHASE 1 "Boot into arch and make the harddisk ready, root has a password. 1st reboot"

Use the latest Arch usb to boot from it

Use the US keyboard otherwise you have to select your keymap manually on “/usr/share /kbd/keymaps” directory
```
loadkeys <yours>
```

### Create partitions for installation

#### I used gdisk to creat partitions. 
#### I created a SHARE partition to have access from both systems at sda3.

```
#:                       TYPE NAME                    SIZE       IDENTIFIER
0:      GUID_partition_scheme                        *121.3 GB   disk0 
1:                        EFI EFI                     209.7 MB   disk0s1 
2:                 Apple_APFS Container disk1         75.0 GB    disk0s2
3:                  Apple_HFS SHARE                   5.1 GB     disk0s3
4:                 Linux Swap                         7.5 GB     disk0s4
5:           Linux Filesystem                         21.5 GB    disk0s5
6:           Linux Filesystem                         12.0 GB    disk0s6
/dev/sda1         (EF00)	121.3 MB/dev/sda2         (AF05)	75 GB
/dev/sda3         (AF05)	5.1 GB
/dev/sda4         (8200)	7.5 GB/dev/sda5         (8300)	21.5 GB
/dev/sda6         (8300)	12.0 GB
```
Make partitions
```mkswap -L SWAP /dev/sda4 mkfs.ext4 -L ROOT /dev/sda5 mkfs.ext4 -L HOME /dev/sda6         
```
### Mount partitions 

```mount /dev/sda5 /mntmkdir /mnt/boot && mount /dev/sda1 /mnt/bootmkdir /mnt/home && mount /dev/sda6 /mnt/homeswapon /dev/sda4
```

### Network

Plugin ethernet cable via adapter.
It usually connects automatically. If necessary run the following: 
```
wifi-menu
```
To check connection:
```
ping yahoo.com
```

### Installation

```
pacstrap /mnt base base-devel bash-completion dialog wpa_suplicant
genfstab -U -p /mnt >> /mnt/etc/fstab
```

### Optimize parameters for SSD, open your fstab file:

```
nano /mnt/etc/fstab
```
Make the following changes for SSD:
As we are using SSD on Macbook. It’s recommended to edit our generated /etc/fstab and add following options to ROOT /dev/sda5 to prolong SSD lifetime:

```
rw,relatime,data=ordered,discard
```

### Pacman update

```
pacman -Sy
nano /etc/pacman.conf
```

un-comment “multilib” repo:
```
[multilib]
Include = /etc/pacman.d/mirrorlist
```

Add Yaourt repository at the bottom:

```
[archlinuxfr]
siglevel = Never
Server = http://repo.archlinux.fr/$arch
```
Alternativelly install yaourt as follows:
```
pacman -S git
git clone https://github.com/Zainpro/install-yaourt
cd install-yaourt
./install-yaourt
```

### Configure your new base system

```
arch-chroot /mnt /bin/bash 
```

Set up locale
```
sudo nano /etc/locale.gen
```
Uncomment desired locals
```
en_US.UTF-8 UTF-8 
en_US ISO-8859-1
```
Generate locales
```
locale-gen
```

### create a settings file to load default language on boot

```
echo LANG=en_US.UTF-8 > /etc/locale.conf
export LANG=en_US.UTF-8
```

### Keymap for the new installation

```
nano /etc/vconsole.conf
```

Insert/edit reqyuired keymap:
```
KEYMAP=us
FONT=lat9w-16
```
Ensure you've chosen the correct keymap otherwise you might be in trouble when login the first time.

### Link the preferred time zone to your localtime link:

```
rm /etc/localtime
ln -s /usr/share/zoneinfo/America/New_York /etc/localtime
```

### Enable the hardware clock

```
hwclock --systohc --utc
```

### Name the system
```echo archbook > /etc/hostname
```
```
nano /etc/hosts
```
Add the new hostname lines

```   127.0.0.1 localhost.localdomain localhost archbook   ::1 localhost.localdomain localhost archbook
```

### Network Manager 
```
pacman -S networkmanager
systemctl enable NetworkManager
```

### Set root password
```
passwd
```
### Bootloader
First we need to find our root partition PARTUUID. Run following command to get PARTUUIDs for each partition:

```
# blkid | awk '{print $1" "$(NF)}'
```
The output is similar to this:
/dev/sda4: PARTUUID="60ce694e-bcc0-47fc-ab4a-df4659a0f1c7" 
/dev/sda5: PARTUUID="d9650118-bc70-4a67-bd31-9283c43e9ad2"

Use the PARTUUID for your root partition, in my case root partition is /dev/sda5.
Now install bootloader:
```
bootctl install 
```

Edit conf as follows:
nano /boot/loader/loader.conf 
```
default arch
timeout 4
editor	0
```
nano /boot/loader/entries/arch.conf
```

Edit it as follows using your own PARTUUID:
```
title	Arch linux
linux	/vmlinuz-linux
initrd	/initramfs-linux.img
options root=PARTUUID=d9650118-bc70-4a67-bd31-9283c43e9ad2 rw ocpi_osi=“!Darwin” quiet
```
### exit chroot:
```exitreboot
```## PHASE 2 "Boot into arch and login as root and make new user, 3rd reboot"login using root password

### Add new user and password

```
useradd -m -g users -G audio,video,network,wheel,storage -s /bin/bash zain
passwd zain
```

### Check internet connection

```
ping yahoo.com
```

If not connected do the following otherwise skip this step:
```ip link
systemctl enable dhcpcd@ens9
ip link set ens9 up```

### Sudo rights

```
pacman -S sudo
EDITOR=nano visudo
```

Unckeck as follows:
```
%wheel ALL=(ALL) ALL
```

Bash completion
```
pacman -S bash-completion
```

Exit root
```
exit
```

## PHASE 3 "Boot into arch, login with username and install your desktop environment"

Check my I3 installation and optimization for MacBook Air.
