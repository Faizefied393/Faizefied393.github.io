# Documentation for Arch Install
## VM set up

1. You will first want the VM to be setup and in order to do this you would would go to the arch linux page to start a download for the ISO 
[Download](https://archlinux.org/download/)


2. In order to download the ISO you would use the checksum so this verifies the files integrity. 

3. First you would right click the ISO and then in order to calculatethe checksum which is SHA256 you would want to do 7-zip.

4. Verifying that there is a match with the check as well as the one given is essential when on the download page. 

5. When opening VM Workstation Pro you will want to create a brand new Virtual Machine.

6. Your selection will be typical and then you will want to hit next. 

7. Select and operating system and appropriate ISO with mine being ("Other Linux 6.x Kernel 64-bit).

8. 20GB is the maximum disk size you want this changed to. 

9. This is done by selecting "Customize Hardware." The memory is also changed to 4 GB. 

10. The number of cores that I picked was "4" and the number or proceessors I picked was "1". 

11. The .vmx file of the VM was opened with notepad and the line 'firmware="efi"' into line 2.


# Arch Setup

1. Start the VM in UEFI Mode. Verify the boot mode using the command:


`ls /sys/firmware/efi/efivars`

2. Checked internet connection with the command:

`ping -c 3 google.com`

3. Enable the Network Time Protocols and allow it to update via the internet:

`timedatectl set-ntp true`

`timedatectl status`

4. Use the `fdisk-1` command in order to list all of the isk drives that are availale. Two partitions will be needed for the hard drive: one boots things into root directory mode and the EEFI mode. The command `fdisk /dev/sda` is needed to begin the hard drives partitioning. In order to create a partition for the boot directory type `n`. In order to partition your numbers and sectors you you choose the primary partition type and click enter. Then for the ginal sector, type "+512M" which sets the size of this partition to 512MB. The partitions name will be `sda1` and then in order to create another partition you do this for the root directory and make sure that the size is the remaining storage by hitting enter for the final sector as well. That partition will go as being called `sda2`. Then to exit and save you would type `w`. 

The (`sda2`) root partition will format to the Ext4 file system. The (`sda1`) boot directory partition will be formatted to the file system that is FAT32. This will be accomplished from the following commands:

`mkfs.ext4 /dev/sda2`

`mkfs.fat -F 32 /dev/sda1`


5. Do the following command to sync the pacman repository:

`pacman -Syy`

6. Install reflector to be able to update the mirrors and sort them by download speed. Add reflector by running:

`pacman -S reflector`

7. If necessary, back up the mirror list with:

`cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.bak`

8. Use the reflector to update the mirror list:

`reflector -c "US" -f 12 -l 10 -n 12 --save /etc/pacman.d/mirrorlist`

9. Mount the root partition with the following syntax:

`mount /dev/sda2 /mnt`

10. Next, use the pacstrap script to install the necessary packages to the bootable partition:

`pacstrap /mnt base linux linux-firmwarre nano`

11. Your installation could take some time depending on your configuration speed. 

12. Now you are generating an fstab file and this is the file that defines the order in which a disk partitions, blocks devices, remote devices, and other data sources are mounted. 

 Create an fstab:

`genfstab -U /mnt >> /mnt/etc/fstab`

13. Use Arch-Chroot and Enter the Mounted Disk as Root by changing the root to the newly installed Arch Linux system having the arch-chroot command:

`arch-chroot /mnt`

14. Set the time zone by listing all of the available time zones which ensures the system clock reflects the accurate local time:

15. List the time zones that are available:

`ls /usr/share/zoneinfo`

16. Listing the subdirectories using the following syntax: 

`ls /usr/share/zoneinfo/America`

17. Find your timezone and do this by making the note of your name, use the ln command to create a symbolic link from your desired timezone to/etc/localtime.

`ln -sf /usr/share/zoneinfo/America/Chicago /etc/localtime`

# In order to set the locale, you set up the locale to determine the language, date, numbering, and currency format for your system. This locale.gen file contains a list of all locales available. 

18. Open the nano file to the find the name of your locale preferrance:

`nano /etc/locale.gen`

Uncomment the name by removing the (#) of your prefferred locale and any other one you would like to use. 

Press control X(Ctrl + X) to exit and type (Y) to save changes. 

19. Then generate the locale configuration file by running the following commands:

`locale-gen`

`echo LANG=en_US.UTF-8 > /etc/locale.conf`

`export LANG=en_US.UTF-8`

20. Set the hostname File and the hostname is a unique identifier for a machine within a network which is especially important in environments with multiple devices helping differentiate between various systems. 

21. Create a hostname file and add your hostname with this syntax:

`echo faiz_arch > /etc/hostname`

22. Create a hosts file via touch command:

`touch /etc/hosts`

23. Add the following content and editing the hosts file:

`127.0.0.1	localhost`
`::1		localhost`
`127.0.1.1	faiz_arch`

24. In order to enable Dynamic Host Configuration Protocol (DHCP), type: 

`systemctl enable dhcpcd`

25. Set up a new root password with the passwd command:

`passwd`

26. Installing GRUB Bootloader on a UEFI system

# This firmware standard replaces the traditional BIOS, which provides an enchanced boot, security, and system management capabilities. 

1. Add the GRUB bootloader packages with the pacman manager:

`pacman -S grub efibootmgr`

2. Create a directory for the EFI partition:

`mkdir /boot/efi`

3. Mount this EFI partition with your created directory:

`mount /dev/sda1 /boot/efi`

4. Now install GRUB

`grub-install --target=x86_64-efi --bootloader-id=GRUB --efi-directory=/boot/efi`

5. Create the GRUB configuration file:

`grub-mkconfig -o /boot/grub/grub.cfg`

# Create a New User/Users for Faiz, Codi, Justin, as well as setting up privelages which is optimal as well as recommended. Having a system with root users does not require authentication for any changes, making it vulnerable and susceptible to failure. 

1. Steps to create a new user:

`pacman -S sudo`

`useradd -m faiz`

`passwd`

2. Create a password(I dont want to disclose mine! Thanks!)

3. Create a password for instructor to reset later: GraceHopper1906

`useradd -m justin`

`passwd`

4. Create a password for instructor to reset later: GraceHopper1906

`useradd -m codi`

`passwd`

5. Adding the users to group of users that grants them specific permissions:

`usermod -aG wheel,audio,video,storage faiz`

`usermod -aG wheel,audio,video,storage justin`

`usermod -aG wheel,audio,video,storage codi`

# Edit the visudo file specifically the line referring to the wheel group and uncomment it:

1. `EDITOR=nano visudo`

2. #%wheel ALL=(ALL:ALL) ALL ->`%wheel ALL=(ALL:ALL) ALL`

3. Save the changes in order to exit the file

Exit-Arch-Chroot Environment and Reboot:

`exit`

`shutdown now`

# Installing the LXQt Desktop in Arch linux

## After reboot, choose Arch Linux from grub. In the Arch Linux prompt, start running the following commands in sequence. These commands install the Xorg server, display manager, LXQt desktop components, controller packages, and additional applications. 

1. Installing Xorg

`sudo pacman -S --needed xorg`

2. Install display manager, lxqt desktop. 

`sudo pacman -S --needed lxqt xdg-utils ttf-freefont sddm`

3. Installing Browser.

`sudo pacman -S firefox`

`firefox &`


# Now enable the display manager and network manager as a service. So that, the next time you log on, hen can run automatically by systemd

1. `systemctl enable sddm`

2. `systemctl enable NetworkManager`

3. Reboot the system using the reboot command 

`reboot`

# Color Coding Terminal
1. The .bashrc file is the file that holds all the settings for the bash terminal. Make a copy of this file to ensure there is a backup in case things go wrong.

`cp .bashrc .bashrc.backup`

2. Use the vim command to edit the .bashrc file:

`vim ~/.bashrc`

3. Create variables for the colors you wish. Insert these variables into the PS1 variable in the places you wish to customize. Relaunch the terminal to view changes. My .bashrc file looks like the following:

`CYAN="\[$(tput setaf 51)\]"`
`RED="\[$(tput setaf 9)\]"`
`RESET="\[$(tput sgr0)\]"`
`PS1="${CYAN}[\u@${RED}\h \W]${RESET}\$ "`

# ZSH Shell

1. Install the zsh package.

`sudo pacman -S zsh`

# After install, run zsh command. A menu will pop up with configuration options. I clicked 1, 2, and 3 and left them all on the default settings. Afterwards, I typed 0 to exit and save. The ZSH shell has been successfully installed.

# Installing SSH

1. Install the openssh package.

`sudo pacman -S openssh`

2. Adding Aliases

permanently add aliases in the .bashrc file. Open the file with editor to start.

`vim ~/.bashrc`

3. Type in the aliases you wish to add add the beggining of the file. Save, exit, and relaunch the terminal to apply all changes. Below, I have listed the aliases I have added in my installation.

`alias c='clear'`
`alias cd..='cd ..'`
`alias grep='grep --color=auto'`
`alias ll='ls -la'`
`alias ls='ls --color=auto'`

## Screenshot for VM
![imgur](https://imgur.com/8fVkEYt.png)
