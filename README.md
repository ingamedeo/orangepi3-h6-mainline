# orangepi3-h6-mainline
Mainline kernel Orange Pi 3 (Allwinner H6) with USB3, WiFi, Ethernet, PCI-E patches. EL2 Hypervisor for quirky PCI-E controller.

### Content
* arm-trusted-firmware: ARM Trusted Firmware-A
* aw-el2-barebone: Allwinner SoCs' 64-bit EL2 barebone hypervisor for PCI-E quirky controller.
* u-boot-2019.04: Mainline u-boot. (not needed)
* u-boot-el1-hyp: Patched u-boot with BL31 and hypervisor support for PCI-Eb quirky controller.
* u-boot-el1-hyp-emmc: Patched u-boot with BL31 and hypervisor support for PCI-Eb quirky controller. Support for boot from EMMC 8GB.
* linux-5.7.4: Mainline kernel + out of tree patches for USB3, WiFi Ethernet, PCI-E controller.

### Instructions

#### arm-trusted-firmware

```bash
make -j8 CROSS_COMPILE=aarch64-linux-gnu- PLAT=sun50i_h6 PRELOADED_BL33_BASE=0x40010000
```
#### aw-el2-barebone

Fix toolchain path in Makefile
```bash
make -j8 ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu-
```

#### u-boot-el1-hyp-emmc

First copy `bl31.bin` from arm-trusted-firmware and `el2-bb.bin` from aw-el2-barebone and rename it to `hyp.bin`.
```bash
make ARCH=arm CROSS_COMPILE=aarch64-linux-gnu- -j4 orangepi_one_plus_defconfig
make -j8 ARCH=arm CROSS_COMPILE=aarch64-linux-gnu-
dd if=u-boot-sunxi-with-spl.bin of=/dev/sdX bs=1024 seek=8
```

#### linux-5.7.4

Compilation:
```bash
make -j8 ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- dtbs modules
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- INSTALL_MOD_PATH=output modules_install
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- INSTALL_MOD_PATH=output headers_install INSTALL_HDR_PATH=output/usr
```

Copy to SD card:
```bash
cp -R linux-5.7.4/arch/arm64/boot/Image /media/ingamedeo/<UUID>/boot/
cp -R linux-5.7.4/arch/arm64/boot/dts/* /media/ingamedeo/<UUID>/boot/dtbs/
cp -R linux-5.7.4/output/lib/ /media/ingamedeo/<UUID>/usr/
cp -R linux-5.7.4/output/usr/ /media/ingamedeo/<UUID>/
```

### References

* https://lkml.org/lkml/2019/4/5/863
* https://github.com/megous/linux/tree/opi3-5.1
* http://linux-sunxi.org/Xunlong_Orange_Pi_3
* https://notsyncing.net/?p=blog&b=2016.orangepi-pc-custom-kernel
* https://forum.armbian.com/topic/13529-a-try-on-utilizing-h6-pcie-with-virtualization/
* https://megous.com/git/linux/commit/?h=orange-pi-5.7&id=9607a07062fdae6e410d32d4807365c5e542b18d


