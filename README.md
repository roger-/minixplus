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
