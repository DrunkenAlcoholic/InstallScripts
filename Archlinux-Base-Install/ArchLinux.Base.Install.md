# Archlinux Base Install



## CHANGE KEYBOARD
Only required for non us keyboard

```bash
find /usr/share/kbd/keymaps/ -type f | more
loadkeys be-latin1
```



## VERIFY BOOT MODE
```bash
ls /sys/firmware/efi/efivars
```
or

```bash
efivar -l
```



## ENABLE WIFI
Not required if using Ethernet
```bash
wifi-menu -o 
```



## TEST CONNECTION
```bash
ping -c 3 google.com
```



## UPDATE SYSTEM CLOCK
replace "Australia/Perth" with your country and city

```bash
timedatectl set-ntp true
timedatectl set-timezone Australia/Perth
```



## CREATE & PARTITION THE DRIVE
list available drives
```bash
lsblk
```

create partitions 
```bash
cfdisk
```

if disk is new you will select type "gpt" for partition table then create the following partitions for EFI

> EFI System partion /dev/sda1 											// With 512M
>
> Swap Partition /dev/sda2 type:swap         						// With 2 x ram
>
> Root Partition /dev/sda3 type:linux system 					// Remainder 
>
> Home Partition Optional /dev/sda4 type:linux system	// Could be on another drive e.g sd**b**1




## FORMAT THE FILESYSTEM
```bash
mkfs.fat -F32 /dev/sda1 
mkswap /dev/sda2
swapon /dev/sda2
mkfs.ext4 /dev/sda3
mkfs.ext4 /dev/sda4
```



## MOUNT THE FILE SYSTEM
Mount our root partition
```bash
mount /dev/sda3 /mnt
```

Create and mount our efi partition

```bash
mkdir /mnt/boot
mount /dev/sda1 /mnt/boot
```

Optional Home, e.g separate drive ect.. 

```bash
mkdir /mnt/home
mount /dev/sda4 /mnt/home
```



## CREATE CLOSEST MIRROR LIST
Replace Australia with your own country
```bash
pacman -Sy
pacman -S reflector
reflector --verbose --country Australia -l 5 --sort rate --save /etc/pacman.d/mirrorlist
```



## INSTALL ARCHLINUX BASE PACKAGES
Note: replace linux package with any kernel of your choice e.g linux-lts, linux-hardened or linux-zen ect..
replace intel-ucode with amd-ucode if using amd processor
```bash
pacstrap -i /mnt base base-devel linux linux-firmware nano intel-ucode
```



## CONFIGURE FSTAB
create the file system table, -U for UUID or -L for lables
```bash
genfstab -U /mnt >> /mnt/etc/fstab
cat /mnt/etc/fstab
```



### CHROOT /MNT
```bash
chroot into system
arch-chroot /mnt
```



## SET YOUR TIME ZONE
Replace /Australia/Perth with your own country and city
```bash
ln -sf /usr/share/zoneinfo/Australia/Perth /etc/localtime
hwclock --systohc
```



## CONFIGURE LANGUAGE AND LOCATION
```bash
nano /etc/locale.gen
```

un-comment your locale, example Australia English
> en_AU.UTF-8 UTF8

Save and exit, then generate locale
```bash
locale-gen
```
```bash
echo LANG=en_AU.UTF-8 > /etc/locale.conf
```

Only required for non us keyboard, example below is setting us keyboard(default)
```bash
echo KEYMAP=us > /etc/vconsole.conf  
```



## SET HOSTNAME
Replace "Archlinux" with host name
```bash
echo Archlinux > /etc/hostname
nano /etc/hosts 
```

insert the following into /etc/hosts, Replacing "Archlinux" with your chosen hostname
```
127.0.0.1 localhost
::1     localhost
127.0.1.1 Archlinux.localdomain Archlinux
```



## SET NETWORK
if using WiFi install the folllowing
```bash
pacman -S wireless_tools wpa_supplicant 
```
Then install networkmanger, optional to install "network-manager-applet" as well if you intend on installing a desktop environment.
```bash
pacman -S networkmanager
systemctl enable NetworkManager
```



## CONFIGURE THE REPOSITORY
```bash
nano /etc/pacman.conf
```

Uncomment the lines: 
```
[multilib]
include = /etc/pacman.d/mirrorlist
```

Update repository 
```bash
pacman -Sy
```



## SET ROOT PASSWORD
```bash
passwd 
```



## INSTALL AND CONFIGURE GRUB BOOTLOADER
This is for Grub bootloader, useful for multiboot, otherwise see system-d bootloader below install bootloader
```bash
pacman -S grub efibootmgr
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
grub-mkconfig -o /boot/grub/grub.cfg
```



## INSTALL AND CONFIGURE SYSTEM-D BOOTLOADER
This is for system-d bootloader, good for single OS, faster than Grub install bootloader
```bash
bootctl --path=/boot install
```

Edit loader.conf
```bash
nano /boot/loader/loader.conf
```

Edit line
> default xxxxxxxxxxxxxx-* to "default arch-*

remove hash from timeout 3 if needed
> "timeout  3"

save and close

Add boot entries
```bash
nano /boot/loader/entries/arch.conf
```

Add the following lines
> title   Arch Linux
>
> linux   /vmlinuz-linux
>
> initrd  /intel-ucode.img  <--- change to amd-ucode if installed during pacstrap
>
> initrd  /initramfs-linux.img
>
> options root=/dev/sda3 rw  <---- replace sda3 with your root partion
>
if you would like to use the UUID instead of /dev/sdax you can print UUID of sda3(in this case) to the arch.conf
```bash
blkid >> /boot/loader/entries/arch.conf
```

Then delete unneeded stuff and keep the UUID, the resulting options line will look like this.
> options root=UUID=XXXXXXXXXXXXXXXXXXXXXXXXXX  rw  <--- replace XXXXX with the UUID from blkid



## ALLOW THE USERS IN WHEEL GROUP TO BE ABLE TO PERFORM ADMINISTRATIVE TASKS WITH SUDO
```bash
useradd -m -g users -G wheel -s /bin/bash drunkenalcoholic
passwd drunkenalcoholic
EDITOR=nano visudo
```

Uncomment the line: 
> %wheel ALL=(ALL) ALL



## UNMOUNT THE PARTITIONS AND REBOOT
```bash
exit
umount -a
reboot
```
or
```bash
shutdown now
```



## ENABLE WIFI
if you installed using wifi-menu then you will need to setup wifi after logging in, Otherwise 
skip this step if you installed using ethernet. Assuming you installed Networkmanager 
and enabled it, its just matter of running nmcli or mntui that comes with Networkmanager.
network-manager-applet can be installed once you have installed a desktop or window manager.

Terminal GUI Wifi
```bash
$ nmtui
```
OR 

List networks
```bash
$ nmcli device wifi list
```

replace "SSID" with your network name, Replace "password" with network password and replace "passwd" 
with your Archlinux password
```bash
$ nmcli device wifi connect SSID password passwd
```



## USER LOG IN
test sudo for user
```bash
$ sudo pacman -Syu
```

get some basic tools for runnning the scripts
```bash
$ sudo pacman -S bash-completion git wget lynx
```



## AUR - Yay
```bash
$ git clone https://aur.archlinux.org/yay.git
$ cd yay
$ makepkg -si
```



## INSTALL XORG
```bash
$ sudo pacman -S xorg-server xorg-xinit xorg-apps xterm 
```



## VIDEO DRIVER
for open source Nvidia
```bash
$ sudo pacman -S xf86-video-nouveau mesa lib32-mesa
```

for newer Nvidia
```bash
$ sudo pacman -S nvidia nvidia-settings nvidia-utils lib32-nvidia-utils
```

for older Nvidia, you need to enable the AUR
```bash
$ sudo yay -S nvidia-390xx nvidia-390xx-utils lib32-nvidia-390xx-utils
```

for newer AMD
```bash
$ sudo pacman -S xf86-video-amdgpu mesa lib32-mesa
```

for older ATI
```bash
$ sudo pacman -S xf86-video-ati mesa lib32-mesa
```

for Intel
```bash
$ sudo pacman -S xf86-video-intel mesa lib32-mesa
```



//============ Install flavour desktop enviroment or window manager ===============//
