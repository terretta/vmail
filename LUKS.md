LUKS disk-encryption
=========

```sh
cfdisk /dev/sda
sda1 -> 200 Mb + Boot
sda2

modprobe dm-crypt
cryptsetup -c camellia -y -s 256 luksFormat /dev/sda2
cryptsetup luksOpen /dev/sda2 lvm

pvcreate /dev/mapper/lvm
vgcreate main /dev/mapper/lvm
lvcreate -l 100%FREE -n root main

mkfs.ext2 /dev/sda1
mkfs.ext4 /dev/mapper/main-root

mount /dev/mapper/main-root /mnt
mkdir /mnt/boot
mount /dev/sda1 /mnt/boot
mkdir /mnt/home

pacstrap /mnt base base-devel grub-bios openssh

genfstab -p -U /mnt >> /mnt/etc/fstab

arch-chroot /mnt

systemctl enable sshd

ln -s /usr/share/zoneinfo/Europe/Paris /etc/localtime
hwclock --systohc --utc

# nano /etc/mkinitcpio.conf
HOOKS="keymap encrypt lvm2 filesystems"

grub-install /dev/sda

# nano /etc/default/grub
GRUB_CMDLINE_LINUX="cryptdevice=/dev/sda2:main"

grub-mkconfig -o /boot/grub/grub.cfg

mkinitcpio -p linux

passwd

exit

umount /mnt/{boot,}

reboot
```
