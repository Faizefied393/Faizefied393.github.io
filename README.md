# Faizefied393.github.io

Arch Linux Installation Documentation-Faiz Tariq

Setting up VM
Navigate to the Arch Linux Download page and download an appropriate ISO.

Use the checksum of the downloaded ISO to verify file integrity. Right-click on the ISO and use 7-zip to calculate the SHA256 checksum. Verify that this checksum matches the given checksum on the download page.

Open VM Workstation Pro and create a new virtual machine. Select “Typical” and hit next. Select the appropriate ISO and Operating System (“Other Linux 6.x Kernel 64-bit). Change the maximum disk size to 20 GB. Select “Customize Hardware…” and change the memory to 4 GB. Change the number of processors to “1” and the number of cores to “4”.

Open the .vmx file of the VM with notepad. Insert the line ‘firmware=”efi”’ into line 2.

Arch Setup
Start the VM in UEFI Mode. Verify the boot mode using the command:


ls /sys/firmware/efi/efivars
Verify internet connectivity using the commands:


ip link
ping google.com
Ensure the system clock is accurate using the command:


timedatectl status
Use the command fdisk -l to view the currently connected hard drives. We will need two partitions for this hard drive: one for booting in UEFI mode and another for the root directory. Use the command fdisk /dev/sda to start partitioning the hard drives. Type n to create a partition for the boot directory. Choose the primary partition type and click enter for the partition number and first sector. For the last sector, type “+500M” to set the size of this partition to 500 MB. This partition will be called sda1. Create another partition for the root directory and let the size equal the remaining storage by clicking enter for the last sector as well. This partition will be called sda2. Type w to exit and save.

The root partition (sda2) will be formatted to the Ext4 file system. The boot directory partition (sda1) will be formatted to the FAT32 file system. This can be accomplished using the following commands:


mkfs.ext4 /dev/sda2
mkfs.fat -F 32 /dev/sda1
Both file systems must also be mounted using the following commands:


mount /dev/sda2 /mnt
mount --mkdir /dev/sda1 /mnt/boot
Essential packages can be installed by executing the pacstrap script. At this point, I also tried installing vim and man-db but encountered errors. I decided to skip this step and try again later.


pacstrap -K /mnt base linux linux-firmware
pacman -S vim man-db #received errors
Generate the fstab file using the following command:

genfstab -U /mnt >> /mnt/etc/fstab
Change root into the newly created system. I attempted installing the packages again and was successfully able to do so. I installed vim as the text editor and network manager and iproute2 for network configuration.


arch-chroot /mnt
pacman -S vim man-db man-pages texinfo 
pacman -S networkmanager iproute2 sudo
Enable the network manager using the command:

systemctl enable NetworkManager
Run the following commands to select the time zone and activate the hardware clock:

ln -sf /usr/share/zoneinfo/America/Chicago /etc/localtime
hwclock --systohc
Use vim to edit the /etc/locale.gen file and uncomment the line en_US.UTF-8 UTF-8. Generate the locales and create the local.conf file. In this file, the LANG variable should be set accordingly.

vim /etc/locale.gen
i #INSERT MODE AND UNCOMMENT LINE
:wq #SAVE AND EXIT
locale-gen
vim /etc/locale.conf
i #INSERT MODE
LANG=en_US.UTF-8
:wq #SAVE AND EXIT
Create the hostname file and insert ArchLinux.


vim /etc/hostname
i # INSERT MODE
ArchLinux
:wq #SAVE AND EXIT
Set the root password using the passwd command.

Boot Loader Setup
I had previously attempted to use the EFISTUB boot loader; however, I ran into many issues that compounded and ended up with me having to restart from scratch. This time, I will be using LXQt as my desktop environment. Get started by installing the following packages:


pacman -S grub efibootmgr
Create an EFI directory in the boot partition. Mount this boot partition to the EFI directory.


mkdir /boot/EFI
mount /dev/sda1 /boot/EFI
Install GRUB to the disk using the following command:



grub-install --target=x86_64-efi --efi-directory=/boot/EFI --bootloader-id=GRUB
Generate the GRUB configuration file using the following command:


grub-mkconfig -o /boot/grub/grub.cfg
Finally, exit the chroot environment and shutdown the virtual machine. Edit the virtual machine settings. Navigate to CD/DVD (IDE) on the sidebar and uncheck the “Connect at power on” box. You can launch into the VM now as the root user.


exit
shutdown now
Desktop Environment
After logging in as root, create a new user with a password for yourself. Edit the sudoers file to give this user sudo permissions. Navigate to the line root ALL=(ALL:ALL) ALL and add your user underneath.


useradd -m faiz
passwd faiz
sudo vim /etc/sudoers
i #INSERT MODE
faiz ALL=(ALL:ALL) ALL
:wq! # "!" needed to override read-only property of file
I have chosen to install the LXQt desktop environment. The following packages will need to be installed:


pacman -S xorg lxqt lxqt-session
Enable the display manager SDDM and restart the VM. You should now see a GUI login page along with the LXQt desktop environment.


systemctl enable sddm.service
shutdown now
Color Coding Terminal
The .bashrc file is the file that holds all the settings for the bash terminal. Make a copy of this file to ensure there is a backup in case things go wrong.


cp .bashrc .bashrc.backup
Use the vim command to edit the .bashrc file:


vim ~/.bashrc
Create variables for the colors you wish. Insert these variables into the PS1 variable in the places you wish to customize. Relaunch the terminal to view changes. My .bashrc file looks like the following:


CYAN="\[$(tput setaf 51)\]"
RED="\[$(tput setaf 9)\]"
RESET="\[$(tput sgr0)\]"
PS1="${CYAN}[\u@${RED}\h \W]${RESET}\$ "
User Codi
User Justin
Create a new user with the following command:


sudo useradd -m codi
sudo useradd -m justin
Create a password for codi and justin using the passwd command:


sudo passwd codi
sudo passwd justin
Set the password to expire so that it must be changed after first login.
GraceHopper1906

sudo passwd -e codi
Add codi and justin to the wheel group to enable sudo permissions.


sudo usermod --append --groups wheel codi
Edit the /etc/sudoers file and uncomment the line with the wheel group.


sudo vim /etc/sudoers
i #INSERT MODE
%wheel ALL=(ALL) ALL
:wq!
ZSH Shell
Install the zsh package.


sudo pacman -S zsh
After the install, run the zsh command. A menu will pop up with many configuration options. I clicked 1, 2, and 3 and left them all on the default settings. Afterwards, I typed 0 to exit and save. The ZSH shell has been successfully installed.

Installing SSH
Install the openssh package.


sudo pacman -S openssh
Adding Aliases
You can permanently add aliases in the .bashrc file. Open the file with an editor to start.


vim ~/.bashrc

Type in the aliases you wish to add towards the beginning of the file. Save, exit, and relaunch the terminal to apply all changes. Below, I have listed the aliases I have added in my installation.

alias c='clear'
alias cd..='cd ..'
alias grep='grep --color=auto'
alias ll='ls -la'
alias ls='ls --color=auto'
