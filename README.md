# Arch linux installation process

## 1. For easier installation process. 
### Enable SSH
```
systemctl start sshd
```
### Set password for root
```
passwd
```
### Get the IP address
```
ip addr
```
### For German keyboard
```
loadkeys de-latin1
```
### Now use any shell that supports copy/past and continue the installation

---

2. ## Now let's create a root (/) partition, format it as ext4 and mount it.
```
echo -e "o\nn\n\n\n\n\na\n\nw\n" | fdisk /dev/sda \
&& mkfs.ext4 /dev/sda1 \
&& mount /dev/sda1 /mnt
```
---

3. ## Now we need to install base packages for running the OS
```
pacstrap /mnt base \
&& genfstab -U -p /mnt >> /mnt/etc/fstab
```
---

4. ## For further installation, we login into the OS
```
arch-chroot /mnt
```
---

5. ## Setup keyboard layout and timezone
```
sed -i 's/#de_DE.UTF-8 UTF-8/de_DE.UTF-8 UTF-8/g' /etc/locale.gen \
&& locale-gen \
&& rm /etc/localtime \
&& ln -s /usr/share/zoneinfo/Europe/Berlin /etc/localtime \
&& hwclock --systohc --utc \
&& localectl set-keymap --no-convert de-latin1
```

---

6. ## Setup the network
### Check the network interface name
#### Display all network interfaces by executing: 
```
ip a
```
#### Then export the network interface name that you want to assign DHCP or static IP.
```
export INTERFACE=???
```

### For dynamic IP (Don't forget to export interface")
```
echo "Description='Local Network'" > /etc/netctl/local-net \
&& echo "Interface="${INTERFACE} >> /etc/netctl/local-net \
&& echo "Connection=ethernet" >> /etc/netctl/local-net \
&& echo "IP=dhcp" >> /etc/netctl/local-net \
&& echo "IP6=stateless" >> /etc/netctl/local-net \
&& netctl enable local-net \
&& systemctl enable dhcpcd
```
### For static IP (Don't forget to export interface, ip, getway and DNS)
#### First export the static IP, GETWAY server and DNS server
```
export IP=???
export GETWAY=???
export DN=???
```
#### Now execute the bellow script to create network file and network service 
```
echo "Description='Local Network'" >> /etc/netctl/local-net \
&& echo "Interface="${INTERFACE} >> /etc/netctl/local-net \
&& echo "Connection=ethernet" >> /etc/netctl/local-net \
&& echo "IP=static" >> /etc/netctl/local-net \
&& echo "IP6=stateless" >> /etc/netctl/local-net \
&& echo "Addres=('"${IP}"')" >> /etc/netctl/local-net \
&& echo "Gateway='"${GETWAY}"' >> /etc/netctl/local-net \
&& echo "DNS=('"${DNS}"')" >> /etc/netctl/local-net \
&& netctl enable local-net \
&& systemctl disable dhcpcd
```

---

7. ## Install neccessary packages
```
pacman -S linux-headers linux-lts linux-lts-headers sudo openssh grub-bios git wget unzip base-devel net-tools gdb 
```

---

8. ## Configure SSH port and enable the service
```
sed -i 's/#Port 22/Port 22/g' /etc/ssh/sshd_config \
&& systemctl enable sshd
```

---

9. ## Install and configure the grub
```
grub-install --target=i386-pc --recheck /dev/sda \
&& grub-mkconfig -o /boot/grub/grub.cfg
```

---

10. ## Create a user with sudo enabled (replace the USER_NAME)
### First export the username
```
export USERNAME=???
```
### Now let's create the user
```
sed -i 's/# %wheel ALL=(ALL) ALL/%wheel ALL=(ALL) ALL/g' /etc/sudoers \
&& useradd -m -g users -G wheel -s /bin/bash ${USERNAME}
&& passwd ${USERNAME}
```

---

11. ## Congratulation we are done. Simply exit and enjoy the Arch linux 
```
exit
umount /mnt
reboot
```
