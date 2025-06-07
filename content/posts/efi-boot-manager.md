---
title: "EFI boot manager"
date: "2024-08-15"
tags: [
    "dual-boot",
    "linux"
]
draft: false
---

 Update EFI Boot Manager. Add new bootloaders for Windows OS and Linux Mint OS 

 ```bash
 # Print available disks (drives) and partitions
lsblk -o name,UUID,partuuid,mountpoint

# Assumption
## Windows OS bootloader is installed on /dev/nvme0n1
## Linux OS bootloader is installed on /dev/nvme1n1

# Add Windows bootloader entry
sudo efibootmgr -c -d /dev/nvme0n1 -p 1 -l \\EFI\\Microsoft\\Boot\\bootmgfw.efi -L "Windows Boot Manager"

# Add Linux Mint bootloader entry
# shimx64.efi supports SecureBoot
sudo efibootmgr -c -d /dev/nvme1n1 -p 1 -l \\efi\\debian\\shimx64.efi -L "Linux Mint"

# Print the bootloader list
sudo efibootmgr

# Output:
## BootCurrent: 0000
## Timeout: 2 seconds
## BootOrder: 0000,0001,0019,0003,0010,0011,0012,0013,0017,0018,001A,001B,001C,001D,001E
## Boot0000* Linux Mint
## Boot0001* Windows Boot Manager
## Boot0003* ubuntu


# Optional: Remove unused bootloaders
## Example: Remove the Boot0003* ubuntu bootloader
## First, inactivate Boot0003 (ID 3)
## sudo efibootmgr -b 3 -A
## Finally, remove Boot0003 (ID 3)
## sudo efibootmgr -b 3 -B

# Print the updated bootloader list 
## sudo efibootmgr


# Update the grub configuration
sudo update-grub 


# Reboot and test

 ```