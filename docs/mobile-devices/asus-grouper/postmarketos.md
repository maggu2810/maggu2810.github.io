Asus Grouper (Google Nexus 7 2012)
===

***postmarketOS related information***

**Author:** *Markus Rathgeb*

---

# General Notes

* first setup u-boot
* after u-boot is installed, install postmarketOS

[postmarketOS Wiki: Asus Grouper](https://wiki.postmarketos.org/wiki/Google_Nexus_7_2012_(asus-grouper)) ([Archive](archive/Google%20Nexus%207%202012%20%28asus-grouper%29%20-%20postmarketOS.pdf))

Ensure correct device is used:

```shell
adb shell
```

```shell
grep androidboot.baseband=unknown /proc/cmdline && echo grouper || echo tilapia

find /sys/devices/ | grep -c max776 && echo You have E1565

find /sys/devices/ | grep -c tps6591 && echo You have PM269
```

Example

```text
~ # grep androidboot.baseband=unknown /proc/cmdline && echo grouper || echo tilapia
tegra_wdt.heartbeat=30 tegraid=30.1.3.0.0 mem=1022M@2048M android.commchip=0 vmalloc=512M androidboot.serialno=015d456d85381609 video=tegrafb no_console_suspend=1 console=none debug_uartport=hsport usbcore.old_scheme_first=1 lp0_vec=8192@0xbddf9000 tegra_fbmem=8195200@0xabe01000 core_edp_mv=0 audio_codec=rt5640 board_info=f41:a00:1:44:2 tegraboot=sdmmc gpt gpt_sector=30785535 androidboot.bootloader=4.23 androidboot.baseband=unknown
grouper

~ # find /sys/devices/ | grep -c max776 && echo You have E1565
451
You have E1565

~ # find /sys/devices/ | grep -c tps6591 && echo You have PM269
0
```

# u-boot

* perform a backup first
  * Guide: [Tegra3 Guide: nvflash NEXUS 7 (and Transformer Jellybeans)](https://androidroot.github.io/pages/guides/tegra3-guide-nvflash-jellybean/) ([Archive](archive/AndroidRoot.Mobi%20-%20Tegra3%20Guide%20nvflash%20NEXUS%207%20%28and%20Transformer%20Jellybeans%29.pdf))
  * The links to the downloads are not working at the time of writing, here you can find them: https://github.com/AndroidRoot/androidroot.github.io/tree/master/download
  * To use nvflash from `nvflash-tools-linux` you need support for 32 bit ELF files (for Fedora 40: `dnf install glibc32 libstdc++.i686`)
  * wheelie source code can be found here, I compiled und used the current (at time of writing) master: https://github.com/AndroidRoot/wheelie
 
* compile u-boot for asus grouper  
  * Guide: [U-Boot for the ASUS/Google Nexus 7 (2012)](https://github.com/u-boot/u-boot/blob/master/doc/board/asus/grouper_common.rst) ([Archive](archive/u-boot_doc_board_asus_grouper_common.rst%20at%20master%20%C2%B7%20u-boot_u-boot%20%C2%B7%20GitHub.pdf))
  * Toolchain: [Arm GNU Toolchain Downloads](https://developer.arm.com/downloads/-/arm-gnu-toolchain-downloads) (I used "x86_64 Linux hosted cross toolchains", "AArch32 bare-metal target (arm-none-eabi)", "arm-gnu-toolchain-13.2.rel1-x86_64-arm-none-eabi.tar.xz")

```
dnf install swig

export PATH=/home/maggu2810/bin/pkgs/arm-gnu-toolchain/arm-gnu-toolchain-13.2.Rel1-x86_64-arm-none-eabi/bin:"${PATH}"
export CROSS_COMPILE=arm-none-eabi-
make grouper_common_defconfig grouper_E1565.config
make
```

* dump SBK
  * [Dump platform key (SBK)](https://openrt.gitbook.io/open-surfacert/common/boot-sequence/tegra-soc-boot-process/fusee-gelee/payloads/dump-platform-key-sbk) ([Archive](archive/Open%20Surface%20RT.pdf))
  * If something went wrong power off the device and enter APX mode again. IIRC you can do some commands only initial on APX mode.

```
git clone --recursive https://github.com/tofurky/tegra30_debrick.git
cd tegra30_debrick
(cd fusee-launcher; make)

python -m venv venv
. venv/bin/activate
pip install pyusb

./fusee-launcher/fusee-launcher.py --tty ./fusee-launcher/dump-sbk-via-usb.bin
```

* u-boot "repart-block.bin"
  * Info:
    * "With nv3p, repart-block.bin is used. It contains BCT and a bootloader in encrypted state in form, which can just be written RAW at the start of eMMC."
    * "Flashing repart-block.bin eliminates vendor restrictions on eMMC and allows the user to use/partition it in any way the user desires."
  * Build
    * Guide: [U-Boot for the ASUS/Google Nexus 7 (2012)](https://github.com/u-boot/u-boot/blob/master/doc/board/asus/grouper_common.rst) ([Archive](archive/u-boot_doc_board_asus_grouper_common.rst%20at%20master%20%C2%B7%20u-boot_u-boot%20%C2%B7%20GitHub.pdf))
    * See:
      * "Process U-Boot", "Processing for the NV3P protocol"
  * Flash
    * Guide: [U-Boot for the ASUS/Google Nexus 7 (2012)](https://github.com/u-boot/u-boot/blob/master/doc/board/asus/grouper_common.rst) ([Archive](archive/u-boot_doc_board_asus_grouper_common.rst%20at%20master%20%C2%B7%20u-boot_u-boot%20%C2%B7%20GitHub.pdf))
    * See:
      * "Flashing U-Boot into the eMMC", "Flashing with the NV3P protocol"

* u-boot usage
  * Boot
    * To boot Linux, U-Boot will look for an extlinux.conf on eMMC.
    * Additionally, if the Volume Down button is pressed while booting, the device will enter bootmenu.
    * Bootmenu contains entries
      * to mount eMMC as mass storage
      * fastboot
      * reboot
      * reboot RCM
      * poweroff
      * enter U-Boot console
      * update bootloader
  * Self Upgrading
    * Place your u-boot-dtb-tegra.bin on the first partition of the eMMC (using ability of u-boot to mount it). Enter bootmenu, choose update bootloader option with Power button and U-Boot should update itself. Once the process is completed, U-Boot will ask to press any button to reboot.

# postmarketOS

## build

* "asus - grouper" moved to "nvidia - tegra-armv7"
  * See: [MR #4606: Nvidia Tegra ARMv7 switch to common packages](https://gitlab.com/postmarketOS/pmaports/-/merge_requests/4606)
* use SXMO for X11
  * [Issues: Phosh on Asus Nexus 7 2012 / grouper](https://gitlab.com/postmarketOS/pmaports/-/issues/1694)

## install 

Pre-requirement: u-boot

* Enter the u-boot bootmenu
* mount internal storage as usb mass storage device
* connect tablet to linux PC
* a new mass storage device should appear

```
dd if=/home/maggu2810/.local/var/pmbootstrap/chroot_native/home/pmos/rootfs/nvidia-tegra-armv7.img of=/dev/sdc bs=1M
```

## customize

### rootfs size

* after system is booted ssh into the system, edit `/boot/extlinux/extlinux.conf`
* add `PMOS_FORCE_PARTITION_RESIZE` kernel parameter to force resizing toofs partition
* reboot
* check `df -h`
* remove parameter again

### SXMO deviceprofile

* [https://sxmo.org/deviceprofile](https://sxmo.org/deviceprofile)
* [https://man.sr.ht/~anjan/sxmo-docs-next/SYSTEMGUIDE.md#writing-a-deviceprofile](https://man.sr.ht/~anjan/sxmo-docs-next/SYSTEMGUIDE.md#writing-a-deviceprofile)
* [https://git.sr.ht/~mil/sxmo-utils/tree/master/item/scripts/deviceprofiles/README.md](https://git.sr.ht/~mil/sxmo-utils/tree/master/item/scripts/deviceprofiles/README.md)
* [https://logout.hu/bejegyzes/hcl/postmarketos_nexus_7_2012_sxmo_2.html](https://logout.hu/bejegyzes/hcl/postmarketos_nexus_7_2012_sxmo_2.html)

```bash
sudo apk add libgpiod
sudo gpuinfo
```

```bash
(

DEVNAME="$(tr '\0' '\n' < /proc/device-tree/compatible | head -n1)"

DPNAME="sxmo_deviceprofile_${DEVNAME}.sh"

DPPATH="/usr/bin/${DPNAME}"

sudo rm -rf "${DPPATH}"

echo '#!/bin/sh
export SXMO_POWER_BUTTON="0:168"
export SXMO_VOLUME_BUTTONS="0:130 0:131"
#export SXMO_TOUCHSCREEN_ID="6"' | sudo tee "${DPPATH}" > /dev/null

sudo chmod 0755 "${DPPATH}"
sudo chown 0:0 "${DPPATH}"

)
```
