**Linux 4.14 device tree blob for [Pineriver Mini-X (Plus) clones](http://linux-sunxi.org/Pineriver_H24) based on the Allwinner A20 SoC.**

Usefull for getting recent Armbian (and possibly other distros) running on these old devices.

See repo for case and board photos.

# Notes

* The DTS file is based on the sun4i-a10-mini-xplus.dts from the mainline kernel, with the analog audio codec enabled and SoC changed to A20. 
* The DTB file was generated from the root of the kernel source tree using:

```sh
cp sun7i-a20-mini-xplus.dts arch/arm/boot/dts/
cpp -nostdinc -I include -I arch  -undef -x assembler-with-cpp  arch/arm/boot/dts/sun7i-a20-mini-xplus.dts dts.tmp
dtc -I dts -O dtb -o sun7i-a20-mini-xplus.dtb dts.tmp
```
* My board has XMD_A20_V1.1 written on it, with a date of 2013.09.10. 
* Cubian for the Cubieboard2 also works, but is outdated (you'll have to use a custom fex file to get the USB port working). It does support NAND installation though.

# Installation

* Install Armbian for the Cubieboard2 on your SD card
* Mount the SD card and replace /boot/dtb/sun7i-a20-cubieboard2.dtb with sun7i-a20-mini-xplus.dtb
* Boot

Note that stock Armbian for the Cubieboard2 *will* work, but without this only the USB OTG port will work (amoung other things). Also if you update your kernel this will probably get overwritten.

# Issues

Everything should work, but RTL8188EU WiFi driver seems to be unstable (when there's no traffic it disconnects). If you keep pinging your router then it should be okay though (e.g. leave `ping 192.168.1.1` running on another terminal).

# Misc

To enable the watchdog timer (e..g to reboot when things freeze or the network goes down), do the following:

1. Install the watchdog service with `sudo apt install watchdog`
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

References:
* https://docs.jethome.ru/en/controllers/linux/howto/watchdog.html
* https://www.supertechcrew.com/watchdog-keeping-system-always-running/ 
