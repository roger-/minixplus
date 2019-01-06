**Linux 4.14 device tree blob for Pineriver Mini-X (Plus) clones based on the Allwinner A20 SOC.**

Usefull for getting Armbian running on these devices.

See repo for case and board photos.

# Overview

The DTS file is based on the sun4i-a10-mini-xplus.dts from the mainline kernel, with the analog audio codec enabled.

The DTB file was generated from the root of the kernel source tree using:

```sh
cp sun7i-a20-mini-xplus.dts arch/arm/boot/dts/
cpp -nostdinc -I include -I arch  -undef -x assembler-with-cpp  arch/arm/boot/dts/sun7i-a20-mini-xplus.dts dts.tmp
dtc -I dts -O dtb -o sun7i-a20-mini-xplus.dtb dts.tmp
```

# Installation

* Install Armbian for the Cubieboard2 on your SD card
* Mount the SD card and replace /boot/dtb/sun7i-a20-cubieboard2.dtb with sun7i-a20-mini-xplus.dtb
* Boot

Note that stock Armbian for the Cubieboard2 *will* work, but without this only the USB OTG port will work (amoung other things). Also if you update your kernel this will probably get overwritten.

# Issues

Everything should work, but 8188eu WiFi driver seems to be unstable. If you keep pinging your router then it should be okay though (e.g. ping 192.168.1.1).
