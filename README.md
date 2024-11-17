
# Arch-Install

A comprehensive guide and scripts for installing Arch Linux, tailored for both beginners and advanced users.

---
## Overview

This repository provides step-by-step instructions and scripts to streamline the installation of Arch Linux. Whether you're new to Linux or an experienced user, this guide will help you set up Arch Linux effectively.

Steps included to Install Arch Linux:
* Step 1: Connect to the internet
* Step 2 : Partitioning disk for installation
* Step 3 : Mount the partitions
* Step 4 : Installing the base system
* Step 5 : Configuring the arch linux system
* Step 6 : Adding a user in Arch Linux
* Step 7 : Update the system and installing required packages
* Step 8 : Enabling system services and daemons
* Step 9 : Bootloader setup
* Step 10 : Post-installation

---
### Prerequisites

- Bootable arch linux pendrive
- Stable internet connection (explained on step number 1)

---
## Installation

### Step 1 : Connect to the internet

 Connect wia the ethernet (preferrable)
#### or

 Connect wia wifi using iwctl tool

- Enter the iwctl interactive shell:
    ```bash
    iwctl
    ```
- Inside iwctl shell,check for available wireless interfaces
    ```bash 
    device list
    ```
    list of available devices will be visible.

- To connect to available network select preferred station with: 

    ```bash
    device wlan0 show
    ```
    ```bash
    station wlan0 get-networks
    ```
- Connect to your wifi network with:
    ```bash
    station wlan0 connect your_wifi_network
    ```
    Enter the passphrase.

- Check the internet connection with
    ```bash
    ping archlinux.org
    ```

---
### Step 2 : Partitioning disk for installation

To check for available drives: 
```bash
lsblk
```

Remove the previous partition with and intialization of new partition can be done by cfdisk:

```bash
cfdisk /dev/sda
```

Create three partitions of EFI boot manager, root partition and swap partition

### Step 3 : Mount the partitions

```
root@archiso ~ # lsblk
NAME    MAJ:MIN RM SIZE RO TYPE MOUNTPOINTS
archiso/airootfs
sda      8:0    0   931.5G  0 disk
├─sda1   8:1    0   512M 0 part
├─sda2   8:2    0    16G  0 part
└─sda3   8:3    0 914.9G  0 part
root@archiso ~ #
```
Here,
- sda1 is EFI boot manager.
- sda2 is swap partition (virtual memory).
- sda3 is root partition.

*Paritions are still not intialized and not mounted, it can be done manually*

Firstly, format root partition by

```bash
mkfs.ext4 /dev/sda3
```

Then, format boot manager partition by,

```bash
mkfs.fat -F 32 /dev/sda1
```

Lastly, create swap partition by

```bash
mkswap /dev/sda2
```
*Its time to mount each partition*

mount root partition with,

```bash
mount /dev/sda3 /mnt
```
 
Boot partition can be mounted by creating mount point directory with following commands:
```bash
mkdir -p /mnt/boot/efi
```
```bash
mount /dev/sda1 /mnt/boot/efi
```

We don't need to mount swap partition, just turn it on with:

```bash 
swapon /dev/sda2
```

And, the final result should look like this

```
root@archiso ~ # lsblk
NAME    MAJ:MIN RM SIZE RO TYPE MOUNTPOINTS
archiso/airootfs
sda      8:0    0   931.5G  0 disk
├─sda1   8:1    0     512M  0 part /mnt/boot/efi
├─sda2   8:2    0      16G  0 part [SWAP]
└─sda3   8:3    0   914.9G  0 part /mnt
root@archiso ~ #
```

### Step 4 : Installing the base system

Use pacstrap script to install the base package, Linux Kernal and firmware:

```bash
pacstrap -K /mnt base linux linux-firmware sof-firmware base-devel grub efibootmgr nano networkmanager
```

After entering this command, base arch system packages with its core will get installed in the root partition. It may take some time according to internet connection.

************************

Information of the file system can be found with genfstab with:

```bash
genfstab /mnt
```
You may see output look something like this:
```
root@archiso ~ # genfstab /mnt
# UUID=4a806a2c-2fcf-4d3f-943e-10dc76aa4b43 /  ext4 rw,relatime                                0 1
# UUID=20DA-C959                             /boot/efi  ufat rw,relatime,fmask=0022,dmask=0022, 0 2
/dev/sda1                                                 shortname=mixed,utf8,errors=remount-ro
# UUID=2d38df14-9f29-4a3e-b2ee-7f09ac6685a7  none       swap defaults                                0 0
/dev/sda2
```
This information is on terminal, but we have to store it on out disk, it can be done via:

```bash
genfstab /mnt > /mnt/etc/fstab
```

*(optional) To ensure whether it is store or not:*

```bash
cat /mnt/etc/fstab
```

The base system is installed and stored on out disk.

### Step 5 : Configuring the arch linux system
We can now enter to our system with chroot
```bash
arch-chroot /mnt
```

- Configuration of the Time Zone
    ```bash
    ln -sf /usr/share/zoneinfo/Asia/Kolkata /etc/localtime
    ```
    check the date and time with "date" command.

    Synchronize the hardware clock with
    ```bash
    hwclock --systohc
    ```
- Configuration of the Localization
    ```bash
    nano /etc/locale.gen
    ```
    I am going to use "en_US.UTF-8 UTF-8", use it by uncommenting the line. And save the file with CTRL+O to save and CTRL+X to exit.
    
    Check the locale with:
    ```bash
    locale-gen
    ```
    You should see output like this:
    ```
    Generating locales...
    en_US.UTF-8... done
    Generation complete.
    ```
    Again, specify the locale in /etc/locale.conf with following commands:
    ```bash
    nano /etc/locale.conf
    ```
    and paste following line on it:
    ```bash
    LANG=en_US.UTF-8
    ```
    once again, you can save it with CTRL+O and CTRL+X to exit.

- Configuration of Hostname
*Hostname : A hostname is a name used to identify a device on a network.*

```bash
nano /etc/hostname
```
Enter desired hostname and exit.
set the password by,
```bash
passwd
```
### Step 6 : Adding a user in Arch Linux
* Adding the user
    ```bash
    useradd -m -G wheel -s /bin/bash user_name
    ```
    Here, 

    *-m means create a home directory.*

    *-G means group is wheel*
    
    *user_name, put your desired username*

#### or

* *(optional) if you don't want to put user in wheel group then,*

    ```bash
    useradd -m user_name
    ```

- Enter the user password with:
    ```bash
    passwd user_name
    ```
- For running sudo commands in user do following changes
    ```bash
    EDITOR=nano visudo
    ```
    inside the /etc/sudoers.tmp, you will find line
    ```
    # Uncomment to allow members of group wheel to execute any command
    # %wheel ALL=(ALL) ALL
    ```
    uncomment it and we can use sudo command.

#### or

* (optional) If you haven't put the user in wheel group, then specify the permission of your username in following line*

    ```
    # User privilege specification
    root ALL=(ALL) ALL
    user_name ALL=(ALL) ALL
    ```

- To run command inside a user,
    ```bash
    su user_name
    ```

### Step 7 : Update the system and installing required packages

* *Make sure you are in home user by writing "whoami", and you will see the name of user that you are currently logged in.*

* *One more way to check the current user is you may see the Shell prompt some thing like*

    ```bash
    [mukund@archlinux ~]$
    ```
    *where mukund is my username and archlinux is my hostname*

* It is good practice to update the system when you login first time in system. You can do it by,

    ```bash
    sudo pacman -Syu
    ```
    and enter your user password.


### Step 8 : Enabling system services and daemons

* Make sure you are in root privilege. It can be done via "sudo su"

* Enable networkmanager with
    ```bash
    systemctl enable NetworkManager 
    ```
* *(optional) You may want to install a login manager like lightdm or sddm, it can be done via the following commands:*
    ```bash
    sudo pacman -S lightdm
    sudo pacman -S lightdm-gtk-greeter
    sudo systemctl enable lightdm
    sudo systemctl start lightdm
    ```
    *If you want a plain and minimal arch install it is advisable to use this login manager. But, we will install it when we set-up our desktop environment in further **step number 10**.*

### Step 9 : Bootloader setup

* We will use Grub as out Bootloader.
* Again, make sure you are in root privileges and also in root directory which can be obtained by "cd".
* You can install grub with following command: 
    ```bash
    grub-install /dev/sda
    ```
* Configure grub with:
    ```bash
    grub-mkconfig -o /boot/grub/grub.cfg
    ```
* Reboot


## Congratulation, we have installed the Arch Linux.

### Step 10 : Post-installation

This step is for those who want to install a Desktop Environment. I will install KDE plasma as my desktop environment.

* Firstly, log in to user and update the system with:
    ```bash
    sudo pacman -Syu
    ```

* To install a complete KDE Plasma Desktop Environment, use following command,

    ```bash
    sudo pacman -S plasma-meta kde-applications
    ```
    *Here,*

     *plasma-meta will only install the core plasma*

     *kde-applications will install all its applications and will provide a complete KDE Plasma experience*

* Installing to the login display manager

    ```bash
    sudo systemctl enable sddm
    ```
    Now onward, at the time of boot, it will use sddm as login manager

    However, to access it now, we can use following command:
    ```bash
    sudo systemctl enable --now sddm
    ```

    ## Welcome to KDE Plasma!!

  ---

  ## 👤 About the Developer

Hello! I'm Mukund Tapaniya, a passionate developer dedicated to creating efficient and user-friendly applications. With a strong Interest in Linux and Artificial Intelligence, I love building systems that improve processes and enhance user experiences.

- **GitHub**: [MukundTapaniya](https://github.com/MukundTapaniya)
- **LinkedIn**: [Mukund Tapaniya](https://in.linkedin.com/in/mukund-tapaniya-296a63250)
- **Email**: [mukundtapaniya47@gmail.com]

Feel free to reach out if you have any questions, suggestions, or just want to connect!
