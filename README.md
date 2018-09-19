# Arch linux installation process

## Enable SSH for copy/past actions
```
systemctl start sshd
passwd
ip addr
```

## For German keyboard
```
loadkeys de-latin1
```

## For deleting entire disk and creating one single partition
```
echo - e "o\nn\np\n1\n\n\nt\nc\na\n1\nw" | fdisk /dev/sda
mkfs.ext4 /dev/sda1
mount /dev/sda1
```

## Install base packages
```
pacstrap /mnt base
genfstab -U -p /mnt >> /mnt/etc/fstab
```

## Login into OS as root for further preparation
```
arch-chroot /mnt
```

## Setup language and timezone
```
sed -i 's/#de_DE.UTF-8 UTF-8/de_DE.UTF-8 UTF-8/g' /etc/locale.gen
locale-gen
rm /etc/localtime
ln -s /usr/share/zoneinfo/Europe/Berlin /etc/localtime
hwclock --systohc --utc
```

## Setup network
### For dynamic IP (Don't forget to replace the interface name)
```
echo "Description='Local Network' \n Interface=(INTERFACE_NAME) \n Connection=ethernet \n IP=dhcp \n IP6=stateless" >> /etc/netctl/local-net
netctl enable local-net
systemctl enable dhcpcd
```
### For static IP (Don't forget to replace the interface name, ip, getway and DNS)
```
echo "Description='Local Network' \n Interface=(INTERFACE_NAME) \m Connection=ethernet \n IP=static \n IP6=stateless \n Addres=('192.168.69.3/24') \n Gateway='192.168.69.1' \n DNS=('192.168.69.2')" >> /etc/netctl/local-net
netctl enable local-net
```

## Set password for root
```
passwd
```

## Create a user with sudo enabled (replace the USER_NAME)
```
sed 's/# %wheel ALL=(ALL) NOPASSWD: ALL/%wheel ALL=(ALL) NOPASSWD: ALL/g' /etc/sudoers > /etc/sudoers.new
export EDITOR="cp /etc/sudoers.new"
visudo
rm /etc/sudoers.new
useradd -m -g users -G wheel -s /bin/bash USER_NAME
passwd USER_NAME
```

## Install neccessary packages
```
pacman -S linux-headers linux-lts linux-lts-headers sudo openssh grub-bios
```

## Configure SSH port and enable the service
```
sed -i 's/#Port 22/Port 22/g' /etc/ssh/sshd_config
systemctl enable sshd
```

## Install and configure the grub
```
grub-install --target=i386-pc --recheck /dev/sda
grub-mkconfig -o /boot/grub/grub.cfg
```

## Exit the base OS and reboot
```
exit
umount /mnt
reboot
```
