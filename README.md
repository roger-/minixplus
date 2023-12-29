# Armbian installation guide for Mini-X clones based on the Allwinner A20 SoC

[These devices](http://linux-sunxi.org/Pineriver_H24) are pretty old and fairly unpopular, but they're supported by Armbian and the latest mainline Linux kernel with minimal issues. Stock Armbian for the Cubieboard 2 will work fairly well, but the device tree needs to be modified to fix a few things, like enabling the second USB port, the LED and disabling some missing hardware.

My particular device is a generic clone with "XMD_A20_V1.1" and date 2013.09.10 on the board. See repo for case and board photos.

# Guide

1. Install the latest [Armbian for Cubieboard 2](https://www.armbian.com/cubieboard-2/) on a micro SD card
1. Mount the card on a Linux machine and create a first run template (needed to have WiFi connect on a headless install)
    ```
    cd <mount directory>/boot/
    sudo cp armbian_first_run.txt.template armbian_first_run.txt
    ```
1. Edit `armbian_first_run.txt` and set `FR_net_wifi_enabled=1` and `FR_net_wifi_ssid` and `FR_net_wifi_key` appropriately
1. Unmount the SD card, place in your device and wait for it to show up on your network
1. Log in via SSH with `root`/`1234` and create a new account
1. Install the modified DTB file from this repo and configure Armbian
    ```
    cd /tmp
    wget https://github.com/roger-/minixplus/raw/master/sun7i-a20-mini-xplus.dtb
    sudo cp sun7i-a20-mini-xplus.dtb /boot/dtb/
    sudo bash -c "echo fdtfile=sun7i-a20-mini-xplus.dtb >> armbianEnv.txt"
    ```
1. Disable unnecessary hardware features by copying `etc/modprobe.d/sunxi.conf` to `etc/modprobe.d/` ([reduces system load](https://forum.armbian.com/topic/7575-k-worker-problem-on-a20-based-boards/))
2. (Optional -- see below) Build a new r8188eu WiFI drive (reduces system load further and improves WiFi stability) 
3. Reboot

You can also login with a keyboard and HDMI display and manually setup the WiFi.

# Notes

* The DTS file is based on the sun4i-a10-mini-xplus.dts and [sun7i-a20-mk808c.dts](https://github.com/torvalds/linux/blob/master/arch/arm/boot/dts/allwinner/sun7i-a20-mk808c.dts) from the mainline kernel, with some modifications. 
* The DTB file was generated from the root of the kernel source tree using:

```sh
cp sun7i-a20-mini-xplus.dts arch/arm/boot/dts/
cpp -nostdinc -I include -I arch  -undef -x assembler-with-cpp  arch/arm/boot/dts/sun7i-a20-mini-xplus.dts dts.tmp
dtc -I dts -O dtb -o sun7i-a20-mini-xplus.dtb dts.tmp
```
* Cubian for the Cubieboard2 also works, but is outdated (you'll have to use a custom fex file to get the USB port working). It does support NAND installation though.
* Recent ArchLinux ARM versions don't seem to work (no HDMI output). Messing with the boot.scr file seems to be necessary.

# Issues

Everything should work, but RTL8188EU WiFi driver seems to be unstable (when there's no traffic it disconnects). If you keep pinging your router then it should be okay though (e.g. leave `ping 192.168.1.1` running on another terminal).

[Alternative wifi drivers](https://github.com/lwfinger/rtl8188eu) may also help.

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
* Test support for integrated microphone
* WiFi stabilization
* Figure out why system load is high

# References
* https://docs.jethome.ru/en/controllers/linux/howto/watchdog.html
* https://www.supertechcrew.com/watchdog-keeping-system-always-running/ 
