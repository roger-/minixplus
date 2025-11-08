# Armbian installation guide for Mini-X clones based on the Allwinner A20 SoC

[These devices](http://linux-sunxi.org/Pineriver_H24) are pretty old and fairly unpopular, but they're supported by Armbian and the latest mainline Linux kernel with minimal issues. Stock Armbian for the Cubieboard 2 will work fairly well, but the device tree needs to be modified to fix a few things, like enabling the second USB port, the LED and disabling some missing hardware.

My particular device is a generic clone with "XMD_A20_V1.1" and date 2013.09.10 on the board. See repo for case and board photos.

# Guide

## Base headless installation

Run the following as root (`su root`):

1. Install the latest [Armbian for Cubieboard 2](https://www.armbian.com/cubieboard-2/) on a micro SD card
1. Mount the card on a Linux machine and create a first run template (needed to have WiFi connect on a headless install)
    ```
    cd <mount directory>/boot/
    cp armbian_first_run.txt.template armbian_first_run.txt
    ```
1. Edit `armbian_first_run.txt` and set `FR_net_wifi_enabled=1` and `FR_net_wifi_ssid` and `FR_net_wifi_key` appropriately
1. Unmount the SD card, place in your device and wait for it to show up on your network
1. Log in via SSH with `root`/`1234` and create a new account

You can also login with a keyboard and HDMI display and manually setup the WiFi. Note that if your WiFi doesn't work then you may have to install an alternate driver.

## Tweaks

Run the following on your device as root:

1. Install the modified DTB file from this repo and configure Armbian
    ```
    cd /tmp
    wget https://github.com/roger-/minixplus/raw/master/sun7i-a20-mini-xplus.dtb
    cp sun7i-a20-mini-xplus.dtb /boot/dtb/
    echo fdtfile=sun7i-a20-mini-xplus.dtb >> armbianEnv.txt
    ```
1. Disable unnecessary hardware features by copying `etc/modprobe.d/sunxi.conf` to `etc/modprobe.d/` ([reduces system load](https://forum.armbian.com/topic/7575-k-worker-problem-on-a20-based-boards/)) then run
   ```
   update-initramfs -u
   ```
1. (Optional -- see below) Build a new r8188eu WiFI driver (reduces system load further and improves WiFi stability) 
1. Reboot

# Build notes

* The DTS file is based on the sun4i-a10-mini-xplus.dts and [sun7i-a20-mk808c.dts](https://github.com/torvalds/linux/blob/master/arch/arm/boot/dts/allwinner/sun7i-a20-mk808c.dts) from the mainline kernel, with some modifications. 
* The DTB file was generated from the root of the kernel source tree using:

```bash
KERNEL_PATH=/tmp/linux-6.12.57

sudo apt-get install flex bison build-essential

# assumes DTS is in current directory
cp sun7i-a20-xmd.dts $KERNEL_PATH/arch/arm/boot/dts/allwinner/

# build
cd $KERNEL_PATH
make ARCH=arm defconfig
make ARCH=arm allwinner/sun7i-a20-xmd.dtb DTC_FLAGS="-@" -j2

# copy DTB
cd -
cp $KERNEL_PATH/arch/arm/boot/dts/allwinner/sun7i-a20-xmd.dtb .
```
* Cubian for the Cubieboard2 also works, but is outdated (you'll have to use a custom fex file to get the USB port working). It does support NAND installation though.
* Recent ArchLinux ARM versions don't seem to work (no HDMI output). Messing with the boot.scr file seems to be necessary.

# WiFi driver

The default RTL8188EU WiFi driver is very unstable and causes a high system load (probably from the `RTW_CMD_THREAD` kernel thread). I've also experienced an issue where it stopped working even after a fresh install. The fix is to install one of the several alternate drivers, such as [this one](https://github.com/aircrack-ng/rtl8188eus).

If you don't have working WiFi on your device then you'll have to somehow install some packages on your Armbian device (build environment, headers and the driver sources). My preferred way is to mount the Armbian installation SD card and `chroot` into it with QEMU, like this:

```bash
sudo apt install qemu-user-static
<mount SD card>
sudo cp /usr/bin/qemu-arm-static sdcard/usr/bin

# set up virtual filesystems
sudo mount -t proc /proc proc/
sudo mount --rbind /sys sys/
sudo mount --rbind /dev dev/

sudo chroot sdcard/ qemu-arm-static /bin/bash

# now in chroot
<download files>
exit

# umount
sudo umount -lf sdcard/
sudo umount sdcard
```

Then boot from the SD card and compile and install the driver. Make sure to blacklist the old driver. 

A binary for kernel 6.1.63 in included in the repo. Install to `/lib/modules/$(KVER)/kernel/drivers/net/wireless/`.

# Watchdog timer

To enable the watchdog timer (e.g. to reboot when things freeze or the network goes down), do the following:

1. Install the watchdog service
   ```
   sudo apt install watchdog
   ```
1. Edit `/etc/default/watchdog` and make sure `run_watchdog=1`
1. Edit `/etc/watchdog.conf` and:
    1. Uncomment `watchdog-device = /dev/watchdog`
    1. Set `watchdog-timeout = 12` (or something <= 16 due to [hardware limitations](https://github.com/torvalds/linux/blob/master/drivers/watchdog/sunxi_wdt.c#L67))
    1. Set `priority = 1` (**important** to [avoid](https://forum.armbian.com/topic/2898-how-to-install-enable-and-start-watchdog-in-h3/?do=findComment&comment=78858) the `cannot set scheduler (errno = 1 = 'Operation not permitted')` error)
    1. Configure the criteria at the bottom of the file (e.g. `ping` or `max-load`)
1. Enable and start the service
    ```
    sudo systemctl enable watchdog
    sudo systemctl start watchdog 
    ```
1. Check if it's running: `sudo service watchdog status`

# Todo
* Cedrus hardware acceleration (e.g. for ffmpeg)

# References
* https://docs.jethome.ru/en/controllers/linux/howto/watchdog.html
* https://www.supertechcrew.com/watchdog-keeping-system-always-running/ 
