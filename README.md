# VinyleOS


git clone --depth 1 https://github.com/torvalds/linux.git

cd linux

make menuconfig

#exit and yes

make -j 4

#bzImage is ready

mkdir /boot-files

cp arch/x86_64/boot/bzImage /boot-files

cd ..

#we are gpng to create busy-box

git clone --depth 1 https://git.busybox.net/busybox

cd busybox

make menuconfig

#Settings - enable "Build static binary (no shared libs)"

#with kernel > 6.6 disable - Networking uilities - disable tc (8.3 kb) NEW

# http://lists.busybox.net/pipermail/busybox-cvs/2024-January/041752.html

make -j 4

#Static linking against glibc, can't use --gc-sections

#Trying libraries: crypt m resolv rt

#Library crypt is not needed, excluding it

#Library m is needed, can't exclude it (yet)

#Library resolv is needed, can't exclude it (yet)

#Library rt is not needed, excluding it

#Library m is needed, can't exclude it (yet)

#Library resolv is needed, can't exclude it (yet)

#Final link with: m resolv


mkdir /boot-files/initramfs

make CONFIG_PREFIX=/boot-files/initramfs install

cd /boot-files/initramfs

vi init

----------------
#!/bin/sh

echo "Welcome to my Linux!"

/bin/sh
----------------

rm linuxrc

chmod +x init

find . | cpio -o -H newc > ../init.cpio

#4936 blocks

apt install syslinux

dd if=/dev/zero of=boot bs=1M count=50

apt install dosfstools

mkfs -t fat boot

syslinux boot

mkdir image

mount boot image

cd /boot-files

cp bzImage init.cpio initramfs/image/

umount initramfs/image


docker cp id://boot-files/initramfs/boot .

qemu-system-x86_64 boot

/bzImage -initrd=/init.cpio
