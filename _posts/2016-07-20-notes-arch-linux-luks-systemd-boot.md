---
layout: post
title: 'Notes: Arch Linux, LUKS+LVM, systemd-boot'
date: 2016-07-20
---

The process for installing an Arch Linux on an encrypted drive, booting from a UEFI machine with systemd-boot.

This assumes a few things:

* No swap partition, or /home partition
* You're already in a liveboot session (via USB or netboot or whatever)
* /boot is also your ESP. (I know this is not great; I only care about my data being encrypted, not the kernel or initramfs.)
* You don't want to use UUID. There's no reason you can't; I just find it annoying.

{% highlight shell %}
# Set up your wifi (or use dhcpcd for a wired connection)
wifi-menu

# Create partitions
# They should look like this:
#   1 500MB EFI/boot partition # type ef00
#   2 100%+ Root partition # type 8300
cgdisk /dev/sda

# Format boot partition (EFI requires fat32)
mkfs.vfat -F32 /dev/sda1

# Encrypt main partition
cryptsetup -c aes-xts-plain64 -y --use-random luksFormat /dev/sda2
cryptsetup luksOpen /dev/sda2 luks

# Set up LVM partition on encrypted partition
pvcreate /dev/mapper/luks
vgcreate vg0 /dev/mapper/luks
lvcreate -l +100%FREE vg0 --name root

# Format LVM partition
mkfs.ext4 /dev/mapper/vg0-root

# Mount up!
mount /dev/mapper/vg0-root /mnt
mkdir /mnt/boot
mount /dev/sda1 /mnt/boot

# Install main system
# dialog and wpa_supplicant are for wireless after the reboot
pacstrap /mnt base base-devel zsh vim git efibootmgr dialog wpa_supplicant

# Install fstab
genfstab -pU /mnt >> /mnt/etc/fstab

# Enter new system
arch-chroot /mnt

# Setup system clock. Your timezone here.
ln -s /usr/share/zoneinfo/Antarctica/McMurdo /etc/localtime
hwclock --systohc --utc

# Set the hostname
echo MYHOSTNAME > /etc/hostname

# Update locale
vim /etc/locale.gen # Uncomment en_US.UTF-8
locale-gen
localectl set-locale LANG=en_US.UTF-8

# Root password
passwd

# Add real user
useradd -m -g users -G wheel,storage,power,audio -s /bin/zsh MYUSERNAME
passwd MYUSERNAME

# Configure mkinitcpio
# Add 'ext4' to MODULES
# Add 'encrypt' and 'lvm2' to HOOKS, before filesystem
vim /etc/mkinitcpio.conf

# Regenerate initrd image
mkinitcpio -p linux

# Setup systemd-boot
bootctl --path=/boot install

# Create systemd-boot configs
echo 'default arch' > /boot/loader/loader.conf
echo 'timeout 5' >> /boot/loader/loader.conf

# Add the following config (without #):
#   title Arch Linux
#   linux /vmlinuz-linux
#   initrd /initramfs-linux.img
#   options cryptdevice=/dev/sda2:vg0 root=/dev/mapper/vg0-root rw
vim /boot/loader/entries/arch.conf

# Done
exit
unmount -R /mnt
reboot
{% endhighlight %}
