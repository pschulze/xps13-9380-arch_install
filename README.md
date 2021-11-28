# XPS 13 9380 - Arch Linux Install

## Starting Point

- [Arch Linux Installation Guide](https://wiki.archlinux.org/title/installation_guide)
- The above guide is excellent and constantly updated, the steps below are areas that tripped me up when following the install guide and what I did to get past them.

## Disk Partitions and File System Selection

- I used `cfdisk` as it's a bit more obvious what you're doing than with good ol' `fdisk`
  - `cfdisk /dev/nvme0n1`
- Make sure to create directories under `/` as needed before mounting partitions
  - E.g. `mkdir /mnt/boot`
- Partition Details:
  |Partition Type     |Size  |Mount Point|File System|Device          |
  |-------------------|------|-----------|-----------|----------------|
  |EFI System         |512M  |`/mnt/boot`|FAT 32     |`/dev/nvme0n1p1`|
  |Linux Swap         |24G   |N/A        |swap       |`/dev/nvme0n1p2`|
  |Linux root (x86-64)|452.4G|`/mnt`     |ext4       |`/dev/nvme0n1p3`|

- Root partition is the remainder of swap space
- Chose not to have a home partition, did that last time and ended up running into issues with space on the root partition for installs
- Lots of swap space because why not
- Commands to format the partitions:

    ```bash
    mkfs.fat -F 32 /dev/nvme0n1p1
    mkswap /dev/nvme0n1p2
    mkfs.ext4 /dev/nvme0n1p3
    ```

## Installing Arch and Extra Packages

```bash
pacstrap /mnt base linux linux-firmware grub efibootmgr intel-ucode vim networkmanager man-db man-pages texinfo
```

- Installs the listed packages to the root partition
- Required for base Arch Install:
  - [base](https://archlinux.org/packages/core/any/base/)
  - [linux](https://archlinux.org/packages/core/x86_64/linux/)
  - [linux-firmware](https://archlinux.org/packages/core/any/linux-firmware/)
- We need a bootloader, in this case GRUB:
  - [grub](https://archlinux.org/packages/core/x86_64/grub/)
  - [efibootmgr](https://archlinux.org/packages/core/x86_64/efibootmgr/)
- Microcode updates, things can get weird without them
  - [intel-ucode](https://archlinux.org/packages/extra/any/intel-ucode/)
- Other nice-to-haves
  - [vim](https://archlinux.org/packages/extra/x86_64/vim/)
  - [networkmanager](https://archlinux.org/packages/extra/x86_64/networkmanager/)
  - man pages
    - [man-db](https://archlinux.org/packages/core/x86_64/man-db/)
    - [man-pages](https://archlinux.org/packages/core/any/man-pages/)
    - [texinfo](https://archlinux.org/packages/core/x86_64/texinfo/)

## Post chroot Steps

### Enabling networkmanager

```bash
systemctl enable NetworkManager.service
```

### Setting Up GRUB

- Install GRUB for Arch

    ```bash
    grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=archlinux_grub
    ```

- Generate the GRUB config file

    ```bash
    grub-mkconfig -o /boot/grub/grub.cfg
    ```

### Done Installing

- Can now exit chroot and restart

  ```bash
  exit
  restart
  ```

## Post-Install

- [General Recommendations](https://wiki.archlinux.org/title/General_recommendations)
