# OpenWrt-A5-V11

### Upgrade Bootloader and Kernel From Factory Firmware

mount /dev/sda1 /mnt
mtd_write write  /mnt/uboot256.img Bootloader
mtd_write write /mnt/firmware.bin Kernel
reboot


### Upgrade Kernel From OpenWrt Firmware

mount /dev/sda1 /mnt
mtd -r write /mnt/firmware.bin firmware


### Restore Factory Default

firstboot
rm -r /overlay/upper/*
rm -r /overlay/*
mtd -r erase rootfs_data


### Install ExtRoot

block detect > /etc/config/fstab; \
   sed -i s/option$'\t'enabled$'\t'\'0\'/option$'\t'enabled$'\t'\'1\'/ /etc/config/fstab; \
   sed -i s#/mnt/sda1#/overlay# /etc/config/fstab; \
   sed -i s#/mnt/sda3#/data# /etc/config/fstab; \
   cat /etc/config/fstab;

echo option force_space >> /etc/opkg.conf

mount /dev/sda1 /mnt ; tar -C /overlay -cvf - . | tar -C /mnt -xf - ; umount /mnt

reboot

opkg update
opkg install luci