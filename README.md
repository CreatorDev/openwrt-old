
![Creator logo](creatorlogo.png)

# Using OpenWrt on Creator (Ci40) Marduk platform

#### For more details about the platform please visit [Creator (Ci40) Marduk](https://community.imgtec.com/platforms/creator-ci40/)
----

## Overview

OpenWrt is a highly extensible GNU/Linux distribution for embedded devices. Instead of trying to create a single, static firmware,
OpenWrt provides a fully writable filesystem with optional package management. This frees you from the restrictions of the application
 selection and configuration provided by the vendor and allows you to use packages to customize an embedded device to suit any application.
For developers, OpenWrt provides a framework to build an application without having to create a complete firmware image and distribution around it.
For users, this means the freedom of full customisation, allowing the use of an embedded device in ways the vendor never envisioned.

Creator is MIPS based platform series from Imagination and Marduk is the latest board under that series. Pistachio is the name of Imagination's SoC based on MIPS.
This guide helps as a quick start but for full details please see the [OpenWrt project wiki](http://wiki.openwrt.org/doc/start).

## Revision History

| revision  | changes from previous revision |
|---------- |------------------------------- |
| 1.0.0     | Initial release                |

## Package Content

The release distribution is structured as documented in the upstream project however
there are some key paths where IMG have added solutions:

| Folder              				| Content                                              							|
| :----               				| :----                                                							|
| target/linux/pistachio			| IMG pistachio SoC based board configs and makefiles   						|
| target/linux/pistachio/config-4.1		| IMG pistachio SoC specific kernel config							        |
| target/linux/pistachio/marduk/profiles/marduk_cc2520.mk	| IMG Marduk platform profile with TI cc2520	|
| target/linux/pistachio/marduk/profiles/marduk_ca8210.mk	| IMG Marduk platform profile with Cascoda ca8210  |
## Getting started

Firstly, to obtain a copy of the (Ci40) Marduk platform supported OpenWrt source code:

 * Sign up for a GitHub account

 * Install Git:  ```` sudo apt-get install git ````

 * Clone the repository: ```` git clone https://github.com/IMGCreator/openwrt.git ````

To simply make a build based on the IMG default config run the following commands:

    $ cd openwrt
    $ ./scripts/feeds update -a
    $ ./scripts/feeds install -a

_Ignore any "WARNING: No feed for package..." from the install feeds step._

Load Marduk platform specific OpenWrt configuration for Pistachio.

    $ make menuconfig

1. Select the "Target System" as IMG MIPS Pistachio

        Target System (Atheros AR7xxx/AR9xxx) --->(X) IMG MIPS Pistachio

2. Check the "Target Profile" is set to Basic platform profile for Marduk

        Target Profile (Basic platform profile for Marduk)  ---> 
            (X) Basic platform profile for Marduk with TI cc2520
            ( ) Basic platform profile for Marduk with Cascoda ca8210

You can also load Marduk platform specific OpenWrt configuration for Pistachio by adding following into .config file.

    $ echo "CONFIG_TARGET_pistachio=y" > .config
    $ echo "CONFIG_TARGET_pistachio_marduk_marduk_cc2520=y" >> .config

Now build OpenWrt in standard way:

    $ make V=s -j1

Once the build is completed, you will find the resulting output i.e. images, dtbs and rootfs at "bin/pistachio", depending upon the selected profile.
- openwrt-pistachio-pistachio_marduk_cc2520-uImage
- openwrt-pistachio-pistachio_marduk_cc2520-uImage-initramfs
- openwrt-pistachio-marduk-marduk-cc2520-rootfs.tar.gz
- pistachio_marduk_cc2520.dtb

## Customising your OpenWrt
You can configure OpenWrt from scratch but it's best to start from a base profile
e.g. the one for the IMG pistachio board as it has some useful defaults.

Simply select the package you need and add into the marduk profile:

    $ vi target/linux/pistachio/marduk/profiles/marduk_cc2520.mk

If you want to add new package, then simply add package name in the list of packages:

    define Profile/marduk
    NAME:=Basic platform profile for Marduk with TI cc2520
    PACKAGES:=kmod-i2c kmod-marduk-cc2520 kmod-sound-pistachio-soc \
                wpan-tools tcpdump uhttpd uboot-envtools \
                alsa-lib alsa-utils alsa-utils-tests \
                iw hostapd wpa-supplicant kmod-uccp420wlan kmod-cfg80211
    endef

Please ensure to remove .config, tmp folders before running make menuconfig.

    $ make menuconfig

Verify that package name is selected. Just save and exit.

## Customising your Linux Kernel

Kernel configuration is saved at target/linux/pistachio/config-4.1
e.g. This the one for the IMG pistachio board as it has some useful defaults.

To customise to suit your requirements use the following command:

    $ make kernel_menuconfig

If you change any option you need and then save & quit GUI, the changed configuration will get written into target/linux/pistachio/config-4.1

For more details please refer to [OpenWrt Build System]("http://wiki.openwrt.org/doc/howto/build")

## Serial Console
Connect the Marduk board to development computer over serial port. Open a serial console on host PC.

- In Windows PuTTY, HyperTerminal, or any other free application can be used.

- In Linux, the simplest option is to use the miniterm.py application that comes bundled with Python.

First work out which device you want, you can see them by running:

    $ ls /dev/ttyUSB*

Then connect to it (remember to choose the correct device listed in above command):

    $ sudo miniterm.py /dev/ttyUSB0 -b 115200

## Boot from USB
As a first step, you need to format USB drive into ext4 parition. Partition your media with one big ext4 partion (use dos partition table)
You can check the partition number by following way:

    $ mount | grep /media/dev/sdx1
    on /media/user/43e3311e-7b95-4693-bfcd-b07fa4590a0d type ext4 (rw,nosuid,nodev,uhelper=udisks2)

Alternatively, you can also check the output of the dmesg command.

Next you need to ensure that it has a DOS partition table containing a Linux ext4 partition.
<b>IMPORTANT! Make sure that you replace /dev/sdx for the right device name, e.g. /dev/sdb</b>

Unmount the filesystem before creating new partition table:

    $ sudo umount /dev/sdx?

Create new partition table:

    $ sudo fdisk /dev/sdx
    > enter 'o' to create a new empty DOS partition table
    > enter 'n' to create a new partition; just press return at every prompt to accept the default values
    > enter 'w' to write the new partition table and exit
    $ sudo mkfs.ext4 /dev/sdx1

Mount the USB drive:

    $ sudo mount /dev/sdx1 /mnt/

To put filesystem on USB you will need 'openwrt-pistachio-marduk-marduk-cc2520-rootfs.tar.gz' which is available at "openwrt/bin/pistachio"

Extract the openwrt-pistachio-marduk-marduk-cc2520-rootfs.tar.gz onto the partition you just created:

    $ sudo rm -rf /mnt/*
    $ sudo tar -xf bin/pistachio/openwrt-pistachio-marduk-marduk-cc2520-rootfs.tar.gz -C /mnt/
    $ sudo umount /mnt/

Run "sync" command to synchronize the data properly on the USB drive:

    $ sync

Connect the USB drive and power on the board.

Press Enter on the [Serial Console](#serial-console) to cancel the autoboot and to get the u-boot prompt. Type following command to boot from USB:

    pistachio # run usbboot

To make boot from USB as default boot method, run the following commands on the u-boot prompt (_ignore any warnings shown_):

    pistachio # setenv bootcmd 'run usbboot'
    pistachio # saveenv

## Boot from SD Card
Similar to the instructions for Boot from USB, you need to format the SD card into ext4 parition. Follow the steps mentioned above by making subtle changes

    $ mount | grep /media/dev/sdc1
    on /media/user/5c0dac14-1d47-4a39-8b06-e27861473670 type ext4 (rw,nosuid,nodev,uhelper=udisks2)

SD media essentially mounts at /dev/sdc1 on a computer.  However, make sure to double-check if it gets somehow mounted differently on your computer, in which case, <b>IMPORTANT! Make sure that you replace /dev/sdc with /dev/sdx for the right device name</b>

Unmount the filesystem before creating new partition table:

    $ sudo umount /dev/sdc1

Create new partition table:

    $ sudo fdisk /dev/sdc
    > enter 'o' to create a new empty DOS partition table
    > enter 'n' to create a new partition; just press return at every prompt to accept the default values
    > enter 'w' to write the new partition table and exit
    $ sudo mkfs.ext4 /dev/sdc1

Mount the SD media:

    $ sudo mount /dev/sdc1 /mnt/

To put filesystem on SD you will need 'openwrt-pistachio-marduk-marduk-cc2520-rootfs.tar.gz' which is available at "openwrt/bin/pistachio"

Extract the openwrt-pistachio-marduk-marduk-cc2520-rootfs.tar.gz onto the partition you just created:

    $ sudo rm -rf /mnt/*
    $ sudo tar -xf bin/pistachio/openwrt-pistachio-marduk-marduk-cc2520-rootfs.tar.gz -C /mnt/
    $ sudo umount /mnt/

Run "sync" command to synchronize the data properly on the SD card:

    $ sync

Connect the SD card in the slot and power on the board.

Press Enter on the [Serial Console](#serial-console) to cancel the autoboot and to get the u-boot prompt. Type following command to boot from SD card:

    pistachio # run mmcboot

To make boot from SD card as default boot method, run the following commands on u-boot prompt (_ignore any warnings shown_):

    pistachio # setenv bootcmd 'run mmcboot'
    pistachio # saveenv

## TFTP Boot
For TFTP boot, you will need TFTP server serving kernel image (uImage), dtb (*.dtb)
and initramfs filesystem.

    $ sudo cp bin/pistachio/openwrt-pistachio-pistachio_marduk_cc2520-uImage-initramfs /tftpboot/uImage
    $ sudo cp bin/pistachio/pistachio_marduk_cc2520.dtb /tftpboot

### Setting up TFTP Server

####Install tftpd package:

    $ sudo apt-get install tftpd

Create /etc/xinetd.d/tftp with following contents:

    service tftp
    {
     protocol        = udp
     port            = 69
     socket_type     = dgram
     wait            = yes
     user            = root
     server          = /usr/sbin/in.tftpd
     server_args     = /tftpboot
     disable         = no
    }

Now restart service:

    $ sudo service xinetd restart

####Configuring U-boot

Now use Serial Console to connect device to host PC. Switch on device and press any key in first 2 seconds. After that you will drop to u-boot console.

To use tftp boot, set the following environment variables

 **Note:**
 - No need to set these environment variables for next boot since these are already saved
 - To get the MAC address, look for a barcode at the bottom side of your board that contains an ID as '0019F5xxxxxx'

1. Set mac address for Ethernet:

        pistachio # setenv ethaddr <00:19:F5:xx:xx:xx>

2. Set Server IP address where TFTP server is running:

        pistachio # setenv serverip <server_ip>

3. Save environment variables:

        pistachio # saveenv

4. Now start tftp boot:

        pistachio # run ethboot

##Boot from Flash

This is the default boot method set on Marduk platform. Either you can copy the openwrt-pistachio-marduk-marduk_cc2520-ubifs.img on the USB drive or you can place the same on TFTP server.
To set up TFTP server on your development PC, refer to [Setting up TFTP Server](#setting-up-tftp-server) section.

1. Init flash device on given SPI bus and chip select:

        pistachio # sf probe 1:0

2. Obtain an IP address (only needed if you are using TFTP server to load the image):

        pistachio # dhcp

3. Define flash/nand partitions:

        pistachio # mtdpart default

4. Erase partition:

        pistachio # nand erase.part firmware0

5. Loading the ubifs image

    Copy the openwrt-pistachio-marduk-marduk_cc2520-ubifs.img to TFTP server (/tftproot) and load from TFTP server:

        pistachio # setenv serverip <development_PC_IP> && tftpboot 0xe000000 openwrt-pistachio-marduk-marduk_cc2520-ubifs.img
OR
    Copy the openwrt-pistachio-marduk-marduk_cc2520-ubifs.img to (ext4 formatted) USB drive and load from USB drive:

        pistachio # usb start && ext4load usb 0 0x0E000000 openwrt-pistachio-marduk-marduk_cc2520-ubifs.img

6. Initialize write to nand device:

        pistachio # nand write 0xe000000 firmware0 ${filesize};

7. Save nand boot environment variables

        pistachio # setenv nandroot "ubi.mtd=firmware0 root=ubi0:rootfs rootfstype=ubifs"
        pistachio # setenv bootcmd 'run nandboot'
        pistachio # saveenv

##System upgrade

Sysupgrade can now be used to flash ubifs images from within OpenWrt:

    root@OpenWrt:/# sysupgrade -v /tmp/openwrt-pistachio-marduk-marduk_cc2520-ubifs.img

You can download the ubifs image from webserver using wget or copy from USB drive.But the image must be put into /tmp as OpenWRT switches to a ramfs to do upgrade.

###Downloading ubifs image from webserver
    root@OpenWrt:/# cd /tmp
    root@OpenWrt:/# wget http://192.168.91.79:8080/openwrt-pistachio-marduk-marduk_cc2520-ubifs.img
(Replace `192.168.91.79:/8080` with IP address:/Portno of your webserver)
###Copying ubifs image from USB drive

    root@OpenWrt:/# mount /dev/sda1 /mnt/cd /tmp
    root@OpenWrt:/# cp /mnt/openwrt-pistachio-marduk-marduk_cc2520-ubifs.img /tmp

The image will be flashed onto the mtd partition that is not in use (firmware0 or firmware1) then uboot is updated to boot from that partition. This requires the firmware0 and firmware1 implementation in uboot with a default environment. The default bootloader binary is based upon U-Boot 2015.07-rc2-299ac94, has this implementation.

Pre-requisite :
U-boot environment variables should be set as follows:

    pistachio # setenv bootcmd 'run dualnandboot'
    pistachio # setenv bootfile uImage
    pistachio # setenv fdtfile pistachio_marduk.dtb

You can also set or print the u-boot environment variables from OpenWrt too:

    root@OpenWrt:/# fw_printenv
    root@OpenWrt:/# fw_setenv bootcmd 'run dualnandboot'
    root@OpenWrt:/# fw_setenv bootfile uImage
    root@OpenWrt:/# fw_setenv fdtfile pistachio_marduk.dtb

Uboot variable bootcount is reset after successful boot.You can cross-check by reading:

    root@OpenWrt:/# cat /sys/devices/platform/*uboot-count/bootcount
    0

You should see the logs on the console as below:

    root@OpenWrt:/tmp# sysupgrade openwrt-pistachio-marduk-marduk_cc2520-ubifs-sysup
    grade.img
    Saving config files...
    Sending TERM to remaining processes ... logd rpcd netifd odhcpd uhttpd dnsmasq awa_bootstrapd awa_clientd awa_serverd ntpd button_gateway_ button_gateway_ device_manager_ sleep ubusd
    Sending KILL to remaining processes ... device_manager_
    Switching to ramdisk...
    [  612.202026] UBIFS (ubi0:0): background thread "ubifs_bgt0_0" stops
    Performing system upgrade...
    Current boot partiton  1
    Writing image to  firmware0
    ubidetach: error!: cannot detach "/dev/mtd4"
           error 19 (No such device)
    ubiformat: mtd4 (nand), size 268435456 bytes (256.0 MiB), 1024 eraseblocks of 262144 bytes (256.0 KiB), min. I/O size 4096 bytes
    libscan: scanning eraseblock 1023 -- 100 % complete
    ubiformat: 1024 eraseblocks have valid erase counter, mean value is 4
    ubiformat: flashing eraseblock 66 -- 100 % complete
    ubiformat: formatting eraseblock 993 -- 96 % com[  628.426599] ubi1: attaching mtd4
    ubiformat: formatting eraseblock 1023 -- 100 % complete
    [  630.259102] ubi1: scanning is finished
    [  630.303104] ubi1: volume 0 ("rootfs") re-sized from 65 to 980 LEBs
    [  630.311869] ubi1: attached mtd4 (name "firmware0", size 256 MiB)
    [  630.318678] ubi1: PEB size: 262144 bytes (256 KiB), LEB size: 253952 bytes
    [  630.326342] ubi1: min./max. I/O unit sizes: 4096/4096, sub-page size 4096
    [  630.333970] ubi1: VID header offset: 4096 (aligned 4096), data offset: 8192
    [  630.341922] ubi1: good PEBs: 1024, bad PEBs: 0, corrupted PEBs: 0
    [  630.348781] ubi1: user volume: 1, internal volumes: 1, max. volumes count: 128
    [  630.356838] ubi1: max/mean erase counter: 8/5, WL threshold: 4096, image sequence number: 1295417059
    [  630.367070] ubi1: available PEBs: 0, total reserved PEBs: 1024, PEBs reserved for bad PEB handling: 40
    [  630.377584] ubi1: background thread "ubi_bgt1d" started, PID 2584
    UBI device number 1, total 1024 LEBs (260046848 bytes, 248.0 MiB), available 0 LEBs (0 bytes), LEB size 253952 bytes (248.0 KiB)
    [  630.491579] UBIFS (ubi1:0): background thread "ubifs_bgt1_0" started, PID 2607
    [  630.623335] UBIFS (ubi1:0): start fixing up free space
    [  630.831119] UBIFS (ubi1:0): free space fixup complete
    [  630.895050] UBIFS (ubi1:0): UBIFS: mounted UBI device 1, volume 0, name "rootfs"
    [  630.903499] UBIFS (ubi1:0): LEB size: 253952 bytes (248 KiB), min./max. I/O unit sizes: 4096 bytes/4096 bytes
    [  630.914678] UBIFS (ubi1:0): FS size: 246333440 bytes (234 MiB, 970 LEBs), journal size 9404416 bytes (8 MiB, 38 LEBs)
    [  630.926538] UBIFS (ubi1:0): reserved for root: 0 bytes (0 KiB)
    [  630.933100] UBIFS (ubi1:0): media format: w4/r0 (latest is w4/r0), UUID D05F05D1-980F-432F-B6DC-DD583FEC53E5, small LPT model
    [  631.028348] UBIFS (ubi1:0): un-mount UBI device 1
    [  631.033687] UBIFS (ubi1:0): background thread "ubifs_bgt1_0" stops
    sysupgrade successful
    [  631.064624] reboot: Restarting system

On successful, it will restart the system and you should following logs on the console:

    ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    BusyBox v1.24.1 (2016-05-02 10:15:17 IST) built-in shell (ash)
    _______                     ________        __
    |       |.-----.-----.-----.|  |  |  |.----.|  |_
    |   -   ||  _  |  -__|     ||  |  |  ||   _||   _|
    |_______||   __|_____|__|__||________||__|  |____|
          |__| W I R E L E S S   F R E E D O M
    -----------------------------------------------------
    DESIGNATED DRIVER (Bleeding Edge, r48138)
    -----------------------------------------------------
    * 2 oz. Orange Juice         Combine all juices in a
    * 2 oz. Pineapple Juice      tall glass filled with
    * 2 oz. Grapefruit Juice     ice, stir well.
    * 2 oz. Cranberry Juice
    $root@OpenWrt:/#
    $root@OpenWrt:/#
    ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

##Configure Network
You can check "ifconfig -a" to check list of interfaces. Ethernet, WiFi and 6loWPAN should be up.

**Note:**

1. 6loWPAN IP has been hardcoded to 2001:1418:0100::1/48. You can change that by editing /etc/config/network script and restarting the network daemon.

        $root@OpenWrt:/# /etc/init.d/network restart

2. You can enable WiFi by default by following below steps:

- set ssid and password for WiFi either at compile time from file target/linux/pistachio/base-files/etc/uci-defaults/config/wireless

        config wifi-iface
            option device       radio0
            option network      sta
            option mode         sta
            option ssid         <XYZ>
            option encryption   psk2
            option key          <Password>

   OR after booting the board, update /etc/config/wireless as above and restart the network by running following command from serial console.

        $root@OpenWrt:/# /etc/init.d/network restart
 
- set default route for WiFi in target/linux/pistachio/base-files/etc/uci-defaults/config/network as

        option 'defaultroute' '1'

### Known Issues:

- Cleaned up kernel patches will be upstreamed soon.
- OPKG support is not implemented.
