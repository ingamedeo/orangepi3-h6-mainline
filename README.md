# orangepi3-h6-mainline
Mainline kernel Orange Pi 3 (Allwinner H6) with USB3, WiFi, Ethernet, PCI-E patches. EL2 Hypervisor for quirky PCI-E controller.

### Content
* arm-trusted-firmware: ARM Trusted Firmware-A
* aw-el2-barebone: Allwinner SoCs' 64-bit EL2 barebone hypervisor for PCI-E quirky controller.
* u-boot-2022.10: Mainline u-boot with native eMMC support.
* u-boot-2019.04: Mainline u-boot. (not needed)
* u-boot-el1-hyp: Patched u-boot with BL31 and hypervisor support for PCI-Eb quirky controller.
* u-boot-el1-hyp-emmc: Patched u-boot with BL31 and hypervisor support for PCI-Eb quirky controller. Support for boot from EMMC 8GB.
* linux-5.7.4: Mainline kernel + out of tree patches for USB3, WiFi Ethernet, PCI-E controller.
* configs: u-boot-2019.04 and linux-5.7.4 configuration files.
* dts: custom dts for the Orange Pi 3 (adds emac support to mainline dts as of 11/16/2022)

### Instructions

#### Build environment
* Ubuntu Linux 18.04.6 LTS

The following packages must be installed:
```
apt-get install build-essential gcc-aarch64-linux-gnu flex bison libssl1.0-dev python2.7-dev python-minimal swig device-tree-compiler
```

#### arm-trusted-firmware

```bash
make -j8 CROSS_COMPILE=aarch64-linux-gnu- PLAT=sun50i_h6 PRELOADED_BL33_BASE=0x40010000
```
#### aw-el2-barebone

```bash
make -j8 ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu-
```

#### u-boot-el1-hyp-emmc

First copy to the u-boot directory the files `bl31.bin` from arm-trusted-firmware and `el2-bb.bin` from aw-el2-barebone and rename the latter to `hyp.bin`.
```bash
cp arm-trusted-firmware/build/sun50i_h6/release/bl31.bin u-boot-el1-hyp-emmc/bl31.bin
cp aw-el2-barebone/el2-bb.bin u-boot-el1-hyp-emmc/hyp.bin
cp configs/u-boot-el1-hyp-config u-boot-el1-hyp-emmc/.config
make -j8 ARCH=arm CROSS_COMPILE=aarch64-linux-gnu-
dd if=u-boot-sunxi-with-spl.bin of=/dev/sdX bs=1024 seek=8
```

#### linux-5.7.4

Compilation:
```bash
cp configs/linux-5.7.4-config linux-5.7.4/.config
make -j8 ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- dtbs modules
make -j8 ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu-
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

rebuild initrd:
```bash
cd /media/ingamedeo/<UUID>/
mount -t proc proc proc/
mount -t sysfs sysfs sys/
mount --bind /dev dev/
chroot /mnt/ /bin/bash
update-initramfs -u -k 5.7.4
```

### Addendum for mainline kernel 5.19.8 or later (no PCIe support)

To get the Orange Pi 3 to boot on mainline kernel with u-boot-2022.10. The mainline kernel now includes support for ethernet, USB3, eMMC (but no PCIe support):
* compile arm-trusted-firmware v2.2, move bl31.bin to the u-boot-2022.10 directory.
* copy dts/sun50i-h6-orangepi-3.dts to u-boot-2022.10/arch/arm/dts/sun50i-h6-orangepi-3.dts
* compile u-boot-2022.10.
* extract ArchLinuxARM-aarch64-latest.tar.gz on the SD card.
* copy u-boot-2022.10/arch/arm/dts/sun50i-h6-orangepi-3.dtb to the SD card at /boot/dtbs/allwinner/sun50i-h6-orangepi-3.dtb

### References

* https://lkml.org/lkml/2019/4/5/863
* https://github.com/megous/linux/tree/opi3-5.1
* http://linux-sunxi.org/Xunlong_Orange_Pi_3
* https://notsyncing.net/?p=blog&b=2016.orangepi-pc-custom-kernel
* https://forum.armbian.com/topic/13529-a-try-on-utilizing-h6-pcie-with-virtualization/
* https://megous.com/git/linux/commit/?h=orange-pi-5.7&id=9607a07062fdae6e410d32d4807365c5e542b18d
* https://linux-sunxi.org/Bootable_eMMC
* https://oftc.irclog.whitequark.org/linux-sunxi/2022-03-16
* https://patchwork.kernel.org/project/linux-arm-kernel/patch/20190411101951.30223-6-megous@megous.com/


