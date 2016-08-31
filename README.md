# OpenWrt-A5-V11

This is an almost step by step tutorial for installing a modified OpenWrt (luci removed usb storage support added) Firmware to A5-V11 Mini Router using the information from these sites:
https://wiki.openwrt.org/toh/unbranded/a5-v11
https://wiki.openwrt.org/doc/howto/extroot

Bootloader image file `uboot256.bin` is taken from this site:
https://github.com/wertwert4pda/rt5350f-uboot

Kernel image file `firmware.bin` is generated using this command:
```
make image PROFILE="A5-V11" PACKAGES="-luci nano blkid block-mount kmod-nls-cp437 kmod-nls-iso8859-1 kmod-nls-utf8 kmod-fs-vfat kmod-fs-ext4 kmod-usb-storage-extras"
```

Image build instructions are taken from this site:
https://wiki.openwrt.org/doc/howto/obtain.firmware.generate 


### Upgrading Bootloader and Kernel From Factory Firmware

Insert a Fat32 formatted USB flash drive to the device USB, connect to it's Wi-Fi and TELNET to 192.168.100.1 (username/password is admin/admin) to enter these commands: 
```
umount /dev/sda1
mount /dev/sda1 /mnt
mtd_write write /mnt/uboot256.img Bootloader
mtd_write write /mnt/firmware.bin Kernel
reboot
```

When you reboot the Wi-Fi is disabled and Ethernet IP becomes 192.168.1.1 so you must connect the device to your computer via ethernet cable and configure your computer with Static IP like 192.168.1.2 and TELNET again.

Enter `passwd` to assign a password to root and exit TELNET, SSH to 192.168.1.1 with your newly created password.


### Installing ExtRoot

First you must regain internet access.

```
nano /etc/config/network
```

Change these lines in the config interface 'lan' section according to your network:
```
option ipaddr '192.168.2.254'
option netmask '255.255.255.0'
```

Add these lines after the netmask line:
```
option gateway '192.168.2.1'
option dns '192.168.2.1'
```

And enter `reboot` again and connect the device to your modem/switch with ethernet cable. SSH to your newly configured IP address and check if you are connected to internet:
```
ping -c 4 google.com
```

Prepare a USB flash drive at a Linux machine and make three partitions:
```
Overlay (1 GiB): primary boot ext4
Swap (512 MiB): primary swap
Data (Rest): primary ext4
```

Plug the USB flash drive and enter this command (all at once) to create fstab and enable partitions:
```
block detect > /etc/config/fstab; \
   sed -i s/option$'\t'enabled$'\t'\'0\'/option$'\t'enabled$'\t'\'1\'/ /etc/config/fstab; \
   sed -i s#/mnt/sda1#/overlay# /etc/config/fstab; \
   sed -i s#/mnt/sda3#/data# /etc/config/fstab; \
   cat /etc/config/fstab;
```

Configure opkg to use extra space and create a mount folder /data for the third partition: 
```
echo option force_space >> /etc/opkg.conf
mkdir /data
```
   
Copy current /overlay to the 1 GiB partition:
```
umount /dev/sda1
mount /dev/sda1 /mnt
tar -C /overlay -cvf - . | tar -C /mnt -xf -
reboot
```

At this step you should see the free space has grown when you use the df command and overlay is mounted as /dev/sda1 when you use the mount command.

Reinstall luci your new and much bigger extroot space:
```
opkg update
opkg install luci
```

You now should have a fully functional OpenWrt with a Web UI (luci) and a lot of space to install whatever you want.

My first recommendations are:
```
opkg install mc e2fsprogs fdisk
opkg install mosquitto mosquitto-client libmosquitto 
```


The following are optional upgrade/restore instructions.

### Upgrading Kernel From OpenWrt Firmware

```
umount /dev/sda1
mount /dev/sda1 /mnt
mtd -r write /mnt/firmware.bin firmware
```


### Restoring Factory Default Settings

```
firstboot
rm -r /overlay/upper/*
rm -r /overlay/*
mtd -r erase rootfs_data
```