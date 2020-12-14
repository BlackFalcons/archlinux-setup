Arch linux EFI tutorial.

How to use this Arch Linux tutorial. {
    1. # tells you that it is a comment.
    2. [] tells you what keys to press.
    3. {} tells you that you are working inside an environment.
    4. () tells you that you need to repeat the previous subroutine you and instruct you if you have to change some of the values.
    5. ! Instructions to do changes, commonly used when requested to input text or edit it.
    6. When a {} instruction is done and if you're in a text editor it means you have to save and exit the editor.
}

# List all keymaps.
ls /usr/share/kbd/keymaps/**/.map.gz | less

# Check if you have internet connection.
ping vg.no

# Enable or disable network time synchronization.
timedatectl set-ntp true

# Show current time settings to check if it is now enabled.
timedatectl status

# Show all partitions.
fdisk -l

# Change partition table.
disk /dev/sda {
    # To create a new partition table.
    g [enter] # To create a new empty GPT partition table made for physical drives like HDD and SSD.

    n [enter] { # To add a new partition.
        [enter] # Select default.
        [enter] # Select default.
        +550M [enter] # Give 550 MB of space to the partition.
        (Repeat twice.
         What to change:
         First +550M -> +2G.
         Second +550M -> default option [enter])
    }

    t [enter] {
        1 [enter] # To select the first partition with 550 MB size. {
            1 [enter] # To select the EFI file system to the selected partition.
            (Repeat once.
             What to change:
             First select 2 [enter] # To select the second partition with 2 GB size.
             Then select 19 [enter] # To select the Linux swap file system for the selected partition.)
        }
    }

    w [enter] # Write the changes we have done on the table to the disk.
}

# Change file system of EFI partition into FAT32.
mkfs.fat -F32 /dev/sda1

# Change file system of swap partition into swap.
mkswap /dev/sda1

# Turn swap on.
swapon /dev/sda2

# Change file system of the main partition into extended filesystem 4.
mkfs.ext4 /dev/sda3

# Mount the main partition to the live image.
mount /dev/sda3 /mnt

# Install base system for Arch linux.
pacstrap /mnt base linux linx-firmware

# Generate file system table.
genfstab -U /mnt >> /mnt/etc/fstab

# Change into root directory of new installation.
arch-chroot /mnt

# Set timezone.
ln -sf /usr/share/zoneinfo/Europe/Oslo /etc/localtime # Changes timezone to your liking.

# Set hardware clock.
hwclock --systhoc

# Installing vim
pacman -S vim

# Setting the locale.gen
vim /etc/locale.gen {
    ! Uncomment the one belonging to you, mine was "nb_NO.UTF-8 UTF-8" as it is for Norway.
}

# Generate locale.
locale-gen

# Setting your hostname.
vim /etc/hostname {
    ! At the start of the file enter your username then save and exit the file.
}

# Creating a hostfile.
vim /etc/hosts {
    ! After the comments go down 2 lines and enter:
    127.0.0.1        localhost
    ::1              localhost
    127.0.1.1        hostname.localdomain   hostname
    ! Replace hostname with the name you put in your /etc/hostname file.
}

# Set root password.
passwd ! Enter a password for the root user.

# Creating a non root user.
useradd USRN ! Replace USRN with a username of your choice.

# Set password on non root user.
passwd USRN ! Replace USRN with the username you selected on your non root account and enter a password for the user.

# Set groups of non root user.
usermod -aG wheel,audio,video,optical,storage USRN ! Replace USRN with the username you selected on your non root account.

# Install sudo command.
pacman -S sudo

# Allow users to use sudo command
visudo {
    ! find "%wheel ALL=(ALL) ALL" and uncomment it by removing #.
}

# Install grub boot loader and other recommended programs..
pacman -S grub efibootmgr dosfstools os-prober mtools

# Create EFI boot directory.
mkdir /boot/EFI

# Mount EFI partition.
mount /dev/sda1 /boot/EFI

# Install grub to partition.
grub-install --target=x86_64-efi --bootloader-id=grub_uefi --recheck

# Create grub config.
grub-mkconfig -o /boot/grub/grub.cfg

# Optinal {
    # Install recommended programs.
    pacman -S networkmanager git

    # Enable networkmanager
    systemctl enable NetworkManager
}

# Unmount mnt.
umount -l /mnt

# Shutdown VM
shutdown now

Edit VM settings {
    ! Enter "settings -> storage" then remove the archlinux iso from storage devices.
}

Virtual box {
    ! Launch your arch linux distro and enter your non root username and password.
}

There you go, you have successfully installed Arch linux on a VM.
