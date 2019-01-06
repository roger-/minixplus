Linux 4.14 Device tree blob for Pineriver Mini-X (Plus) clones based on the Allwinner A20 SOC.

Usefull for getting Armbian running on these devices.

See the repo for case and board photos.

# Overview

The DTS file is based on the sun4i-a10-mini-xplus.dts from the mainline kernel, with the analog audio codec enabled.

The DTB file was generated from the root of the kernel source tree using:

```sh
cpp -nostdinc -I include -I arch  -undef -x assembler-with-cpp  arch/arm/boot/dts/sun7i-a20-mini-xplus.dts  dts.tmp
dtc -I dts -O dtb -o sun7i-a20-mini-xplus.dtb dts.tmp
```

# Installation

* Install Armbian for the Cubieboard2 on your SD card
* Mount the SD card and replace /boot/dtb/sun7i-a20-cubieboard2.dtb with sun7i-a20-mini-xplus.dtb

Note that stock Armbian for the Cubieboard2 *will* work, but without this only the USB OTG port will work (amound other things).
