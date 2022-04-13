# Connect to the internet
	iwtcl
	device list
	station wlan0 scan
	station wlan0 get-networks
	station wlan0 connect CT2A-2B1

# Update the system clock
	timedatectl set-ntp true

# Partitioning
	fdisk /dev/sdb
	...

# Formating
	mkfs.fat -F32 /dev/sdb5
	mkswap /dev/sdb6
	mkfs.btrfs -f /dev/sdb7

# Mounting
	mount -t btrfs -o ssd,noatime,autodefrag,compress=zstd:5 /dev/sdb7 /mnt
	btrfs subvolume create /mnt/@
	btrfs subvolume create /mnt/@home
	btrfs subvolume create /mnt/@snapshots
	umount /mnt
	mount -t btrfs -o ssd,noatime,autodefrag,compress=zstd:5,subvol=@ /dev/sdb7 /mnt
	mkdir -p /mnt/{boot/efi,home,.snapshots}
	mount -t btrfs -o ssd,noatime,autodefrag,compress=zstd:5,subvol=@home /dev/sdb7 /mnt/home
	mount -t btrfs -o ssd,noatime,autodefrag,compress=zstd:5,subvol=@snapshots /dev/sdb7 /mnt/.snapshots
	mount /dev/sdb5 /mnt/boot/efi
	swapon /dev/sdb6

# Install base system
	pacstrap /mnt ...

## Base
	base base-devel linux linux-headers linux-firmware

## Tools
	vim git xdg-utils xdg-user-dirs-gtk intel-ucode btrfs-progs dialog mtools dosfstools reflector ntfs-3g

## Internet
	networkmanager network-manager-applet wpa_supplicant

## Bluetooth
	bluez bulez-utils

## Sound
	pipewire

## WebRTC
	xdg-desktop-portal xdg-desktop-portal-gnome

## Grub
	grub brub-btrfs os-prober efibootmanager

## nvidia
	nvidia nvidia_modeset nvidia_uvm nvidia_drm

## GNOME
	gnome-shell gnome-control-center eog evince file-roller gdm gnome-keyring gnome-shell-extensions chrome-gnome-shell nautilus gnome-console (AUR) gnome-text-editor (AUR)

# Generate fstab
genfstab -U /mnt >> /mnt/etc/fstab
cat /mnt/etc/fstab

# Chroot
arch-chroot /mnt

# Time zone
ln -sf /usr/share/zoneinfo/Asia/Ho_Chi_Minh /etc/localtime
hwclock

# Localization
vim /etc/locale.gen
// Uncomment en_US.UTF-8 UTF-8
locale-gen
vim /etc/locale.conf
// LANG=en_US.UTF-8

# Network configuration
echo MyArch > /etc/hostname
// Add entries
127.0.0.1	localhost
::1	localhost
127.0.0.1	MyArch.localdomain	MyArch

# Enable services
systemctl enable networkmanager
systemctl enable bluetooth.service
systemctl enable gdm

# Install YAY


# mkinitcpio
vim /etc/mkinitcpio.conf
//Edit line 7:
MODULES=(btrfs nvidia nvidia_modeset nvidia_uvm nvidia_drm)
mkinitcpio -p linux

# Install grub
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=Archlinux
grub-mkconfig -o /boot/grub/grub.cfg

# Root password
passwd









