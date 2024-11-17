
# Arch-Install

A comprehensive guide and scripts for installing Arch Linux, tailored for both beginners and advanced users.

## Overview

This repository provides step-by-step instructions and scripts to streamline the installation of Arch Linux. Whether you're new to Linux or an experienced user, this guide will help you set up Arch Linux effectively.

### Prerequisites

- Bootable arch linux pendrive
- Stable internet connection (explained on step number )

## Installation

### Step 1 - connect to the internet

 Connect wia the ethernet
### or

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
    device wlon0 show
    ```
    ```bash
    staion wlan0 get-networks
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

### Step 2 - Partitioning disk for installation

To check for available drives: 
```bash
lsblk
```

Remove the previous partition with and intialization of new partition can be done by cfdisk:

```bash
cfdisk /dev/sda
```

Create three partitions of EFI boot manager, root partition and swap partition

```
root@archiso ~ # lsblk
NAME    MAJ:MIN RM SIZE RO TYPE MOUNTPOINTS
archiso/airootfs
sda      8:0    0   32G  0 disk
├─sda1   8:1    0   100M  0 part
├─sda2   8:2    0    4G  0 part
└─sda3   8:3    0 27.9G  0 part
root@archiso ~ #
```
Here,
- sda1 is EFI boot manager.
- sda2 is swap partition (virtual memory).
- sda3 is root partition.

Paritions are still not intialized and not mounted, it can be done manually

