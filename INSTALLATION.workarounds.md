# Yumei Installation Instructions

I think it's just better to run lots of commands with the root user because i might forgotten to include some sudo
in the commands listed bellow lmao

## Creating the disks

Asumming that your lsblk looks like:

```
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
nbd0         43:0    0    60G  0 disk 
├─nbd0p1     43:1    0   512M  0 part 
└─nbd0p2     43:2    0  59.5G  0 part /mnt
```

you could do:

```sh
sudo mkdir -pv /mnt
sudo mount /dev/nbd0p2 /mnt
sudo mkdir -pv /mnt/boot
sudo mount /dev/nbd0p1 /mnt
```

## Setup

I recommend to store all the yumei related source code in a isolated place:

```sh
mkdir -pv ~/workspace/yumei/
cd !$
```

## Running akari

Akari is gonna bootstrap your yumei installation (give password for the creating of the yumei user when needed).

```sh
git clone https://github.com/Yumei-Linux/akari.git
cd akari
cargo build --release
sudo install -Dvm755 ./target/release/akari /usr/bin
sudo akari --yumei-root=/mnt
```

> Give passwd for the yumei user.

## Installing yumei chroot

This tool lets you enter in the yumei system without having to care about
virtual kernel filesystems.

```sh
git clone https://github.com/yumei-linux/yumei-chroot.git
cd yumei-chroot
sudo make PREFIX=/usr install
```

## Installing the refine script

The refine scripts refines the yumei installation with the needed files after
the akari phase.

```sh
git clone https://github.com/yumei-linux/refine.git
cd refine
sudo make PREFIX=/mnt/usr install # Install it inside the mounted drive
```

## Running refine inside the chroot

```sh
sudo yumei-chroot /mnt
refine
exit
exit
sudo yumei-chroot /mnt
```

> The double exit and the re-login with yumei-chroot isn't needed, but if you wanna get the right prompt yeah lol

## Getting yumi

In order to get yumi, you firstly have to build some deps that should have been automatically built by akari
but that's wip lol

- zlib
- bzip2
- perl
- openssl

> you could refer to pages in the lfs guide (the ones in the chapter 8) on how to build them, also note that the sources are already downloaded at /sources

After building those packages you could just do:

```sh
exit # exit the chroot
git clone https://github.com/yumei-linux/yumi.git
cd yumi
cargo build --release
sudo install -Dvm755 ./target/release/yumi /mnt/usr/bin
```

Now install the initial yumi-packages

```sh
git clone https://github.com/yumei-linux/yumi-packages.git /mnt/var/yumi
```

## Configuring make_opts and ninja_jobs

If you wanna use -jX and X jobs for the ninja spawning in yumi you could do this:

```sh
sudo yumei-chroot /mnt
mkdir -pv /etc/yumi
# you might consider changing -j10 with the appropiate one for your system.
echo '-j10' > /etc/yumi/make_opts
echo '10' > /etc/yumi/ninja_jobs
```

## Installing the base and the base-full packages in order to get a fully functional yumei linux install

```sh
yumi grab system/base system/base-full
```

> Now you just have to wait a loooooong time.

## Configuring ca-certificates in order to get sources downloading working inside the chroot

You have to download the ca-certificate pkg dependencies first, this will be automatically
configured by akari in the future.

```sh
exit # exit the chroot
wget http://anduin.linuxfromscratch.org/BLFS/other/make-ca.sh-20170514
mv -v ./make-ca.sh* /mnt/sources/ca-certificates
wget http://anduin.linuxfromscratch.org/BLFS/other/certdata.txt
mv -v ./certdata.txt /mnt/sources/ca-certificates
yumei-chroot /mnt
```

## Building ca-certificates

```sh
yumi grab security/ca-certificates
```

## Installing wget and curl as testing packages for the network sources downloading

This pkgs are in this phase used just for testing if the sources downloading works.

```sh
yumi grab net/wget net/curl
```

Then try them

```sh
wget https://www.google.com/
rm -rvf ./index.html
curl https://www.google.com/
```

## Making the system bootable

### Fstab

First you have to write the fstab

```sh
cd /etc
vim fstab # vim comes preinstalled
```

example fstab

```
LABEL=YUMEI_BOOT    /boot   vfat    defaults    0 2
LABEL=YUMEI_ROOT    /       ext4    defaults    1 1
```

## Installing the linux kernel

Refer to the lfs documentation in order to build a linux kernel from source.

> remember to put it at /boot/

## Installing grub

Grab the `system/grub-bios` package by using yumi.

```sh
yumi grab system/grub-bios
```

Then install the grub

```sh
cd
grub-install /dev/nbd0 --target=i386-pc
grub-mkconfig -o /boot/grub/grub.cfg
```
