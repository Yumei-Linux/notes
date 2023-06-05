# Yumei Preliminary Installation Guide

## Creating filesystems

- /dev/sda1 - 512M linux efi system or just bootable
- /dev/sda2 - XG linux
- /dev/sda3 - XG swap (maybe)

## Formatting drives

```sh
mkfs.fat -F 32 /dev/sda1
fatlabel /dev/sda1 YUMEI_BOOT
mkfs.ext4 /dev/sda2 -L YUMEI_ROOT
mkswap /dev/sda3 -L YUMEI_SWAP
swapon /dev/disk/by-label/YUMEI_SWAP
```

## Mounting filesystems

```sh
mkdir -pv /mnt
mount /dev/disk/by-label/YUMEI_ROOT /mnt
```

## Running akari

> Refer to the akari repo to see installation.

```sh
akari --yumei-root=/mnt
```

> Akari will install the first version of the yumi's database at /mnt/var/yumi, to then be able to run yumi sync you would have to build git

## Running refine and yumei-chroot

> Install yumei-chroot and install refine at /mnt/usr/bin

```sh
yumei-chroot /mnt
$ (yumei chroot) refine
```

## Installing refine-yumi-setup

> Install the refine-yumi-setup tool at /mnt/usr/bin

```
$ (yumei chroot) refine-yumi-setup
Installing yumi's deps...
Installing yumi...
Installing first version of yumi's database...
```

## Installing base packages with yumi

```sh
yumi grab base base-full
# Lots of output
```

## What's next?

Coming soon.