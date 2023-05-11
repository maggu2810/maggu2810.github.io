OnePlus 6: Fedora
===

***Create Fedora images for OnePlus 6***

**Author**: *Markus Rathgeb*

---

# Current short instructions

```shell
export IMG_PATH="${PWD}/oneplus-enchilada-fedora.img"
rm -rf "${IMG_PATH}"
truncate -s 5G "${IMG_PATH}"


export DEV_IMG="$(sudo losetup -P -f "${IMG_PATH}" -b 4096 --show)"


sudo parted -s "${DEV_IMG}" mktable msdos
sudo parted -s "${DEV_IMG}" mkpart primary ext2 2048s 256M
sudo parted -s "${DEV_IMG}" mkpart primary 256M 100%
sudo parted -s "${DEV_IMG}" set 1 boot on

export DEV_BOOT="${DEV_IMG}p1"
export DEV_ROOT="${DEV_IMG}p2"

sudo mkfs.ext4 -O ^metadata_csum -F -q -L pmOS_root -N 100000 "${DEV_ROOT}"
sudo mkfs.ext2 -F -q -L pmOS_boot "${DEV_BOOT}"


mkdir -p mnt
sudo mount "${DEV_ROOT}" mnt
sudo mkdir -p mnt/boot
sudo mount "${DEV_BOOT}" mnt/boot


podman export fedora-aarch64-device.c | sudo tar -C mnt/ -xp


cp mnt/boot/boot.img boot.img


sudo umount mnt/boot
sudo umount mnt


sudo losetup -d "${DEV_IMG}"


img2simg oneplus-enchilada-fedora.{img,simg}
mv oneplus-enchilada-fedora.{simg,img}


fastboot erase --slot=all system
fastboot erase userdata
fastboot erase --slot=all boot
fastboot erase dtbo
fastboot flash boot --slot=all boot.img
fastboot flash userdata oneplus-enchilada-fedora.img
```

# Introduction

After receiving my OnePlus 6 I first tried the postmarketOS distribution.
It worked out of the box.

For flashing I used this commands:

```shell
fastboot erase --slot=all system
fastboot erase userdata
fastboot erase --slot=all boot
fastboot erase dtbo
fastboot flash boot --slot=all /tmp/postmarketOS-export/boot.img
fastboot flash userdata /tmp/postmarketOS-export/oneplus-enchilada.img
```

I enabled the ssh daemon using the terminal on the phone (`sudo service sshd start`) and could connect to
it (`ssh user@172.16.42.1` - default password "147147").

As postmarketOS provied a kernel and a rootfs (boot partition and root partition to be more detailed) my idea has been
first use the postmarketOS kernel and replace the postmarketOS rootfs by Fedora rootfs.

To not change the kernel I use the same "userdata format" as postmarketOS.

For the OP6 userdata contains a msdos partition table with two partitions, one for boot and one for root (see [https://wiki.postmarketos.org/wiki/Partition_Layout](https://wiki.postmarketos.org/wiki/Partition_Layout)).

I did the same keep boot as it is, filled a rootfs with Fedora rootfs and copied firmware and modules from the postmarketOS rootfs.

It worked a little bit but not as expected.

So, I did some further diagnosis...

I read the postmarketOS initramfs and I like it approach. It does some good things:

* setup the USB-C jack to act as gadget, assign IP address and provide a DHCP server 
* find and mount filesystems
* execute hooks from a initramfs-extra that is stored in the /boot partitions
* add some information on the framebuffer
* switch root

Using the hooks, we could takeover the initramfs logic and integrate all the steps after the hooks to a custom hook and provide information to the framebuffer...

Just an example to get rid of the pbsplash to show kernel messages again (if enabled):

```shell
$ cat /hooks-extra/001.sh
```

```shell
#!/bin/sh

. /init_functions.sh
source_deviceinfo

killall pbsplash 2>/dev/null
while pgrep pbsplash >/dev/null; do
  sleep 0.01
done
```

After the switch_root has been executed, I have to analyze why the bootup stucks.

So, let's enable ssh daemon (we already have a kernel set IP address, so no need to wait for network target or similar) as early as possible:

Add to the rootfs something like that

```shell
cat /etc/systemd/system/sshd-simple.service
```

```text
[Unit]
Description=OpenSSH server daemon
Documentation=man:sshd(8) man:sshd_config(5)

[Service]
Type=notify
EnvironmentFile=-/etc/sysconfig/sshd
ExecStart=/usr/sbin/sshd -D $OPTIONS
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process
Restart=on-failure
RestartSec=42s
```

```shell
[unpriv@fedora ~]$ ls -lah /etc/systemd/system/*/sshd-simple.service 
lrwxrwxrwx 1 root root 22 May  7  2023 /etc/systemd/system/basic.target.wants/sshd-simple.service -> ../sshd-simple.service
lrwxrwxrwx 1 root root 22 May  7  2023 /etc/systemd/system/getty.target.wants/sshd-simple.service -> ../sshd-simple.service
```

and ensure that there are some server keys in `/etc/ssh`.

# Software

## Create Packages

Not found on https://packages.fedoraproject.org/

* pd-mapper (done)
    * https://github.com/andersson/pd-mapper.git
        * copr: jforbes/fedora-x13s
        * copr: ahalaney/pd-mapper
    * 'pd-mapper' is the reference implementation for Qualcomm's Protection Domain mapper service.
    * It is required for userspace applications to access the various remote processors (Wi-Fi, modem, sensors...) on
      recent Qualcomm SoCs using the QRTR protocol.
    * ?? pd-mapper is some other funky shit. embedded platform go brrr ??
* rmtfs (todo)
    * https://github.com/andersson/rmtfs
        * copr: missing
    * Qualcomm Remote Filesystem Service Implementation
    * ?? rmtfs provides a service the modem uses to interact with the filesystem ??
* qrtr (done)
    * https://github.com/andersson/qrtr
        * copr: jforbes/fedora-x13s
    * Qualcomm IPC Router support
    * Userspace reference for net/qrtr in the Linux kernel
* tqftpserv (todo)
    * https://github.com/andersson/tqftpserv.git
        * copr: missing
    * Trivial File Transfer Protocol server over AF_QIPCRTR
    * ?? tqftpserv allows browsing the filesystem (?) and ??
* qbootctl (done)
    * https://gitlab.com/sdm845-mainline/qbootctl
        * copr: missing
    * A port of the Qualcomm Android bootctrl HAL for musl/glibc userspace.
    * Qualcomm bootctl HAL for Linux
    * qbootctl is some tool that talks to qcom api to tell it that the boot was a success
    * without it device will sometimes go into coredump mode
        * https://gitlab.gnome.org/maggu2810/op6.git
        * enable system service op6/src/mark-boot-successful.service
* kernel package to sync postmarketOS kernel
* linux-firmware-oneplus6
  * https://gitlab.com/sdm845-mainline/firmware-oneplus-sdm845

## Check Build Configuration etc.

* ModemManager
    * grrrr, modemmanager on arch for some reason disables support for qcom devices
    * even more grrr, arch ships too old deps for mm from main
    * yay, one more thing to get modem working:
    * modemmanager doesn't support multi-sim,
    * so op6 needs a weird script to run at boot for provisioning the sim card
    * https://gitlab.freedesktop.org/mobile-broadband/ModemManager.git
        * https://gitlab.gnome.org/maggu2810/op6.git
        * (op6/src/op6-provision-simcard.sh +) enable service op6/src/op6-provision-simcard.service

# Instructions

TEMPORARY TO REMEMBER MYSELF!!

I am using podman to create the Fedora rootfs content and some adjustments, so the [Containerfile](https://github.com/maggu2810/op6-containers/blob/main/fedora/device/Containerfile) is of interest, too.


```shell
oneplus-enchilada:/home/user# parted -a opt /dev/sda17
GNU Parted 3.6
Using /dev/sda17
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted) p                                                                
Model: Unknown (unknown)
Disk /dev/sda17: 54.1GB
Sector size (logical/physical): 4096B/4096B
Partition Table: msdos
Disk Flags: 

Number  Start   End     Size    Type     File system  Flags
 1      8389kB  256MB   247MB   primary  ext2         boot
 2      256MB   54.1GB  53.9GB  primary  ext4

(parted)                                                                 
```

https://www.cyberciti.biz/faq/linux-backup-restore-a-partition-table-with-sfdisk-command/

```shell
oneplus-enchilada:/home/user# sfdisk -d /dev/sda17
label: dos
label-id: 0xd7b6d7b7
device: /dev/sda17
unit: sectors
sector-size: 4096

/dev/sda17p1 : start=        2048, size=       60416, type=83, bootable
/dev/sda17p2 : start=       62464, size=    13153467, type=83
```

## rootfs

### create image storage

```shell
export IMG_PATH="${PWD}/oneplus-enchilada-fedora.img"
rm -rf "${IMG_PATH}"
truncate -s 5G "${IMG_PATH}"
```

### integrate image

```shell
export DEV_IMG="$(sudo losetup -P -f "${IMG_PATH}" -b 4096 --show)"
#losetup --json --list
```

### create partitions

```shell
sudo parted -s "${DEV_IMG}" mktable msdos
sudo parted -s "${DEV_IMG}" mkpart primary ext2 2048s 256M
sudo parted -s "${DEV_IMG}" mkpart primary 256M 100%
sudo parted -s "${DEV_IMG}" set 1 boot on

export DEV_BOOT="${DEV_IMG}p1"
export DEV_ROOT="${DEV_IMG}p2"

sudo mkfs.ext4 -O ^metadata_csum -F -q -L pmOS_root -N 100000 "${DEV_ROOT}"
sudo mkfs.ext2 -F -q -L pmOS_boot "${DEV_BOOT}"
```

### mount partitions

```shell
mkdir -p mnt
sudo mount "${DEV_ROOT}" mnt
sudo mkdir -p mnt/boot
sudo mount "${DEV_BOOT}" mnt/boot
```

### fill rootfs

#### fedora container

```shell
podman export fedora-aarch64-device.c | sudo tar -C mnt/ -xp
```

#### OLD: fedora image

This has been an alternative to the container approach.

Kept for information only!

**obsolete** because the adjustment done in Containerfile must be done again

```shell
# download https://kojipkgs.fedoraproject.org/compose/38/latest-Fedora-38/compose/Spins/aarch64/images/Fedora-Phosh-38-1.6.aarch64.raw.xz

export F_IMG_PATH="/home/maggu2810/Downloads/op6/fedora/Fedora-Phosh-38-1.6.aarch64.raw"
sudo kpartx -a "${F_IMG_PATH}"
export F_DEV_ROOT="/dev/mapper/loop0p3"
mkdir -p f_mnt
sudo mount "${F_DEV_ROOT}" f_mnt
sudo cp -ax f_mnt/root/* mnt/
sudo umount f_mnt
sudo kpartx -d "${F_IMG_PATH}"
```

#### OLD: generic

**obsolete** done on container creation

```shell
sudo cp -avx /home/maggu2810/.local/var/pmbootstrap/chroot_rootfs_oneplus-enchilada/boot/* mnt/boot/
sudo cp -avx /home/maggu2810/.local/var/pmbootstrap/chroot_rootfs_oneplus-enchilada/lib/modules/* mnt/lib/modules/
sudo cp -avx /home/maggu2810/.local/var/pmbootstrap/chroot_rootfs_oneplus-enchilada/lib/firmware/* mnt/lib/firmware/
```

### OLD: initramfs

**obsolete** there is no need for

Keep the information to remember how to modify the extra and add hooks

```shell
rm -rf initramfs
mkdir initramfs
(cd initramfs; zcat ../mnt/boot/initramfs | cpio -idmv)

rm -rf initramfs-extra
mkdir initramfs-extra
(cd initramfs-extra/; zcat ../mnt/boot/initramfs-extra | cpio -idmv)

cp -ax hooks-extra/ initramfs-extra/

(cd initramfs-extra; find . | cpio -o -c -R root:root | gzip -9 > ../initramfs-extra-ng)
sudo mv initramfs-extra-ng mnt/boot/initramfs-extra 
```

### umount partitions

```shell
sudo umount mnt/boot
sudo umount mnt
```

### de-touch image

```shell
sudo losetup -d "${DEV_IMG}"
```

### create android sparse image

```shell
img2simg oneplus-enchilada-fedora.{img,simg}
mv oneplus-enchilada-fedora.{simg,img}
```

# ...

```shell
curl -O "https://gitlab.com/sdm845-mainline/pmtools/-/raw/main/utils/mkbootimg.sh"
chmod +x mkbootimg.sh

./mkbootimg.sh \
  -d mnt/boot/dtbs/qcom/sdm845-oneplus-enchilada.dtb \
  -r mnt/boot/initramfs \
  -k mnt/boot/vmlinuz \
  -o "${PWD}/mainline-boot.img"

# no kernel command line customization ATM using -c e.g. -c root=/dev/sda17 (if using different image layout)
```

```
mkbootimg.sh: Appending dtb "./mnt/boot/dtbs/qcom/sdm845-oneplus-enchilada.dtb" to image "./mnt/boot/vmlinuz"
mkbootimg 	--base '0x00000000' 	--kernel_offset '0x00008000' 	--ramdisk_offset '0x01000000' 	--tags_offset '0x00000100' 	--pagesize '4096' 	--second_offset '0x00f00000' 	--cmdline " " 	--os_patch_level 2019-09-21 	--kernel /tmp/kernel-dtb -o '/home/maggu2810/pmos/mainline-boot.img' --ramdisk mnt/boot/initramfs
mkbootimg.sh: Boot image at /home/maggu2810/pmos/mainline-boot.img


####
cmdline using
mkbootimgs.sh without "-c ..."
and
uname -a: Linux fedora 6.2.0-sdm845 #2-postmarketos-qcom-sdm845 SMP PREEMPT Tue Mar 14 00:05:59 UTC  aarch64 GNU/Linux

androidboot.verifiedbootstate=orange androidboot.keymaster=1 root=PARTUUID=19cc22c8-45f4-27de-fa05-9f48999ec5d3 androidboot.bootdevice=1d84000.ufshc androidboot.fstab_suffix=default androidboot.serialno=083b54e9 androidboot.baseband=msm msm_drm.dsi_display0=dsi_samsung_sofef00_m_cmd_display: androidboot.slot_suffix=_a skip_initramfs rootwait ro init=/init androidboot.dtb_idx=-1347440721 panel_type=black androidboot.mode=normal androidboot.recoveryreason=000 androidboot.project_name=17819 androidboot.project_codename=enchilada ddr_manufacture_info=Samsung ddr_row0_info=16 androidboot.hw_version=22 androidboot.rf_version=32 androidboot.prj_version=0 androidboot.platform_id=321 androidboot.platform_name=SDM845 androidboot.startupmode=pon1 androidboot.enable_dm_verity=1 androidboot.at_location=factory androidboot.power_cut_test=0 androidboot.secboot=enabled androidboot.battery.absent=false androidboot.rpmb_enable=true androidboot.type=normal androidboot.product.hardware.sku=3 androidboot.init_log_level=49 androidboot.cust=0 androidboot.prmec=true androidboot.opcarrier=none androidboot.bootcount=1919247457
####

[unpriv@fedora ~]$ cat /etc/fstab 
/dev/mapper/sda17p2 / ext4 rw,relatime 0 0

```

## Create

```shell
echo "${HOME}/.local/var/pmbootstrap"'
edge
oneplus
enchilada
y
unpriv
none
n
none
y
C.UTF-8
oneplus6
n
y' | pmbootstrap init
pmbootstrap install
pmbootstrap export
install android-tools for simg2img
simg2img /tmp/postmarketOS-export/oneplus-enchilada.img oneplus-enchilada.img
export IMG_PATH="${PWD}/oneplus-enchilada.img"
mkdir root
sudo mount /dev/loop0p2 root
sudo mount /dev/loop0p1 root/boot
sudo tar czpf oneplus-enchilada.tgz -C root/ .
sudo umount root/boot root/
rmdir root
rm oneplus-enchilada.img
```

# Notes

## Partitions

This list has been saved using postmarketOS on my phone (as root):

```shell
blkid | sort -g
```

```text
/dev/dm-0: LABEL="pmOS_boot" UUID="e577a746-23fc-4dfb-a855-c81e5ceede8b" BLOCK_SIZE="4096" TYPE="ext2"
/dev/dm-1: LABEL="pmOS_root" UUID="3c266d0a-d0b6-45fb-a367-d28056dec1d4" BLOCK_SIZE="4096" TYPE="ext4"
/dev/sda10: PARTLABEL="reserve1" PARTUUID="698a851a-f33f-cd61-0d1b-826603a77ca4"
/dev/sda11: PARTLABEL="reserve2" PARTUUID="48ca4c46-75c1-3f5b-7c4a-846026908c6f"
/dev/sda12: PARTLABEL="config" PARTUUID="ed8f57c6-2bb7-4622-73ff-042e5dce6fb0"
/dev/sda13: LABEL="/" UUID="74ab2787-89b4-5f32-9f21-2c085107ac1e" BLOCK_SIZE="4096" TYPE="ext4" PARTLABEL="system_a" PARTUUID="19cc22c8-45f4-27de-fa05-9f48999ec5d3"
/dev/sda14: LABEL="/" UUID="c070087f-00b9-5294-8485-a9542d23a5c4" BLOCK_SIZE="4096" TYPE="ext4" PARTLABEL="system_b" PARTUUID="fb77c341-58db-78b5-3212-a3e8c80d2363"
/dev/sda15: LABEL="odm" UUID="1778e345-2c06-433d-84ca-5b0aabc12474" BLOCK_SIZE="4096" TYPE="ext4" PARTLABEL="odm_a" PARTUUID="08aec4a5-b82b-c2cf-dca7-fcca675360b2"
/dev/sda16: LABEL="odm" UUID="1778e345-2c06-433d-84ca-5b0aabc12474" BLOCK_SIZE="4096" TYPE="ext4" PARTLABEL="odm_b" PARTUUID="196a8d0f-c73a-078f-9c4a-620bf7c882b0"
/dev/sda17: PTUUID="d7b6d7b7" PTTYPE="dos" PARTLABEL="userdata" PARTUUID="d7201bd9-a5a4-7413-705e-42ddb9906ca8"
/dev/sda1: PARTLABEL="ssd" PARTUUID="90b16030-9d6e-8b41-f47f-575a2b601c11"
/dev/sda2: UUID="97df8ae2-99d3-45a6-a09e-8c7a1691247c" BLOCK_SIZE="4096" TYPE="ext4" PARTLABEL="persist" PARTUUID="8c782723-4bd3-5914-e3b7-90a2de933a04"
/dev/sda3: PARTLABEL="misc" PARTUUID="77e2d4ef-a9ee-4801-35c6-c9dfbee4ccda"
/dev/sda4: PARTLABEL="param" PARTUUID="162b5d9c-7558-db06-6f18-892bccae8cad"
/dev/sda5: PARTLABEL="keystore" PARTUUID="5478c133-c748-702a-e3d4-7b7a980b0da4"
/dev/sda6: PARTLABEL="frp" PARTUUID="c94d3031-3b43-5dd0-fc74-d45f925462a5"
/dev/sda7: LABEL="op2" UUID="05e2189f-0ff6-4219-a490-b6f4323cbc88" BLOCK_SIZE="4096" TYPE="ext4" PARTLABEL="op2" PARTUUID="2d46c473-838e-4d97-a824-f447f846075f"
/dev/sda8: PARTLABEL="oem_dycnvbk" PARTUUID="3be0090d-52d4-8b6b-5366-27079fc3b574"
/dev/sda9: PARTLABEL="oem_stanvbk" PARTUUID="bc19edc6-8498-60de-980d-269f4925fb7b"
/dev/sdb1: PARTLABEL="xbl_a" PARTUUID="9dd5b108-3d9d-9df6-273a-3fdeabb0dd58"
/dev/sdb2: PARTLABEL="xbl_config_a" PARTUUID="0b8aa997-f093-80de-d0eb-b822f3a4bbaa"
/dev/sdc1: PARTLABEL="xbl_b" PARTUUID="9795e8c7-a4bc-37ee-ec57-f2404a97b407"
/dev/sdc2: PARTLABEL="xbl_config_b" PARTUUID="67af4168-1602-8b9f-3e4f-576106777ed3"
/dev/sdd1: PARTLABEL="ALIGN_TO_128K_1" PARTUUID="26ffa336-0b0d-07fc-8acb-a95637524e33"
/dev/sdd2: PARTLABEL="cdt" PARTUUID="a0ec5e99-cf0e-3337-b43c-59b5f04cc668"
/dev/sdd3: PARTLABEL="ddr" PARTUUID="b18ff3f4-0b09-bf50-f8a7-74f85d8c6cfd"
/dev/sde10: PARTLABEL="keymaster_a" PARTUUID="c15b5503-05e2-74ea-9dd1-3ddaa1933664"
/dev/sde11: PARTLABEL="boot_a" PARTUUID="1dd3a986-997c-0c48-1d1b-b0d0399f3153"
/dev/sde12: PARTLABEL="cmnlib_a" PARTUUID="4be9c484-fead-b4df-028b-c42f19bb771d"
/dev/sde13: PARTLABEL="cmnlib64_a" PARTUUID="7a676289-f499-4849-2e3e-6110c72f8b3c"
/dev/sde14: PARTLABEL="devcfg_a" PARTUUID="cb0d0e64-6ccf-412c-a1a1-6a6f6899c0cf"
/dev/sde15: PARTLABEL="qupfw_a" PARTUUID="0c29a859-40c6-0351-85f0-40bcb67a9e70"
/dev/sde16: LABEL="vendor" UUID="0c70056c-5d72-53f7-93e4-c56a1b61e40a" BLOCK_SIZE="4096" TYPE="ext4" PARTLABEL="vendor_a" PARTUUID="b5e4e456-0c89-c7aa-4671-dfcfa58c4a50"
/dev/sde17: PARTLABEL="vbmeta_a" PARTUUID="ea36dbc9-3ef3-8033-289a-8fd65ad70cd8"
/dev/sde18: PARTLABEL="dtbo_a" PARTUUID="88acca40-bf3f-05c5-c64c-57d0b94ef345"
/dev/sde19: PARTLABEL="storsec_a" PARTUUID="e2d23828-7af0-2a38-9e54-dc992d9ec299"
/dev/sde1: PARTLABEL="aop_a" PARTUUID="18a9eb6c-34ba-778a-378e-24a2ca53423d"
/dev/sde20: PARTLABEL="LOGO_a" PARTUUID="7d09acd3-fdc7-24a3-3ef1-7231c8557dc5"
/dev/sde21: PARTLABEL="fw_4j1ed_a" PARTUUID="7984bbfb-9c60-969a-511b-1927110bd37b"
/dev/sde22: PARTLABEL="fw_4u1ea_a" PARTUUID="0de21a2d-16ed-a2a5-3628-27cbf5f6a7b6"
/dev/sde23: PARTLABEL="fw_ufs3_a" PARTUUID="99cfddd1-c334-fbd3-4592-dd3dae42d7e4"
/dev/sde24: PARTLABEL="fw_ufs4_a" PARTUUID="79c101ca-486a-4761-d1fd-42d4a90ecd8f"
/dev/sde25: PARTLABEL="fw_ufs5_a" PARTUUID="ba5b3351-d3c7-334a-5d4f-6bc6ee18c76e"
/dev/sde26: PARTLABEL="fw_ufs6_a" PARTUUID="959d46fc-62d6-1b81-a3c7-4782878bf939"
/dev/sde27: PARTLABEL="fw_ufs7_a" PARTUUID="8aef2687-45eb-6104-822c-ba3ebcda03aa"
/dev/sde28: PARTLABEL="fw_ufs8_a" PARTUUID="7dca79b5-2813-87dd-7241-2538fa3f6809"
/dev/sde29: PARTLABEL="aop_b" PARTUUID="ef572bf2-e170-61e2-636d-1e00d9bdb9ad"
/dev/sde2: PARTLABEL="tz_a" PARTUUID="c468c1af-db1c-568f-8ec7-53b735416d25"
/dev/sde30: PARTLABEL="tz_b" PARTUUID="8eec337d-8ffd-bcef-45dd-fbbaef420af8"
/dev/sde31: PARTLABEL="hyp_b" PARTUUID="c2f15698-daec-9aa2-a097-f7c0ec244fa1"
/dev/sde32: SEC_TYPE="msdos" UUID="00BC-614E" BLOCK_SIZE="4096" TYPE="vfat" PARTLABEL="modem_b" PARTUUID="d80fff11-faf5-e403-1e07-b3f92da5df14"
/dev/sde33: SEC_TYPE="msdos" UUID="00BC-614E" BLOCK_SIZE="4096" TYPE="vfat" PARTLABEL="bluetooth_b" PARTUUID="6c8f5520-3b25-f1aa-113e-5444ec6b7bc3"
/dev/sde34: PARTLABEL="mdtpsecapp_b" PARTUUID="c093098a-c29c-67d5-87cc-b823325f500c"
/dev/sde35: PARTLABEL="mdtp_b" PARTUUID="5d694b4a-8561-d7fd-4f88-192d902bf8d1"
/dev/sde36: PARTLABEL="abl_b" PARTUUID="f83f5a71-7460-d50f-af91-e30535309ccb"
/dev/sde37: LABEL="dsp" UUID="af32c008-2a39-7e5b-a5dc-201456d93103" BLOCK_SIZE="4096" TYPE="ext4" PARTLABEL="dsp_b" PARTUUID="79ba0936-0c74-64ae-e995-8c041e2d684b"
/dev/sde38: PARTLABEL="keymaster_b" PARTUUID="ede27ba9-4c06-b49f-e30e-cd081c12cefa"
/dev/sde39: PARTLABEL="boot_b" PARTUUID="45105095-3847-4657-51f2-2a0144550453"
/dev/sde3: PARTLABEL="hyp_a" PARTUUID="ba8bafdb-7f50-1e8f-346a-c13abf300941"
/dev/sde40: PARTLABEL="cmnlib_b" PARTUUID="cdb50a31-5cb3-646a-5ffa-37f1fc7cf02d"
/dev/sde41: PARTLABEL="cmnlib64_b" PARTUUID="6f4fa8ad-0733-6d2a-4641-3335db470550"
/dev/sde42: PARTLABEL="devcfg_b" PARTUUID="d501d92e-854e-6cdf-bd8a-b29c5303b24f"
/dev/sde43: PARTLABEL="qupfw_b" PARTUUID="209724bc-ee5d-f227-2c4d-a3e8e9364db1"
/dev/sde44: PARTLABEL="vendor_b" PARTUUID="dbe58f5c-303e-d453-ef76-91c9c88cbd7e"
/dev/sde45: PARTLABEL="vbmeta_b" PARTUUID="f0c2dc1c-9b74-9dc4-b0da-4887611bb8b7"
/dev/sde46: PARTLABEL="dtbo_b" PARTUUID="a82e816e-fe23-6ac6-1015-ae0344a28b28"
/dev/sde47: PARTLABEL="storsec_b" PARTUUID="9538581c-f700-75cd-d11c-ab5d4301790e"
/dev/sde48: PARTLABEL="LOGO_b" PARTUUID="262aaa7b-61c0-b7f2-9526-4ce1e664e96f"
/dev/sde49: PARTLABEL="fw_4j1ed_b" PARTUUID="76de293a-76e8-f885-d33c-c9caa5f95104"
/dev/sde4: SEC_TYPE="msdos" UUID="00BC-614E" BLOCK_SIZE="4096" TYPE="vfat" PARTLABEL="modem_a" PARTUUID="761d9a2e-73d1-bf89-52dd-f16d65f3949f"
/dev/sde50: PARTLABEL="fw_4u1ea_b" PARTUUID="164fd6d9-9ca4-478b-1703-5a764b214c3c"
/dev/sde51: PARTLABEL="fw_ufs3_b" PARTUUID="9f5d63d8-77bd-dbc0-61f2-fa2e0b338a03"
/dev/sde52: PARTLABEL="fw_ufs4_b" PARTUUID="251cb7ad-053c-fa41-115f-767116727e4e"
/dev/sde53: PARTLABEL="fw_ufs5_b" PARTUUID="6dc25d07-16a8-0e5f-eca6-e6bf3881195d"
/dev/sde54: PARTLABEL="fw_ufs6_b" PARTUUID="b10083d8-de96-7a82-f72a-12baaa0d3751"
/dev/sde55: PARTLABEL="fw_ufs7_b" PARTUUID="9dd01a5a-d0d0-572a-6b93-e93c1c7c65fc"
/dev/sde56: PARTLABEL="fw_ufs8_b" PARTUUID="630122ae-5505-88cb-f65b-0e694810a868"
/dev/sde57: PARTLABEL="minidump" PARTUUID="b5df8fd2-39ab-e2b1-0123-0d6d07948f31"
/dev/sde58: PARTLABEL="boot_aging" PARTUUID="b3d11f25-855a-524f-b3af-961ad2a937ab"
/dev/sde59: LABEL="op1" UUID="58a6b23e-5ce2-4d69-881b-69d3b061564b" BLOCK_SIZE="4096" TYPE="ext4" PARTLABEL="op1" PARTUUID="2190addd-4b30-9c0e-7b43-44e1c943850f"
/dev/sde5: SEC_TYPE="msdos" UUID="00BC-614E" BLOCK_SIZE="4096" TYPE="vfat" PARTLABEL="bluetooth_a" PARTUUID="690f015d-1f6a-9041-7a71-41457afcc4c3"
/dev/sde60: PARTLABEL="sec" PARTUUID="40b72c38-8636-49d4-7c38-dd1274ab928b"
/dev/sde61: PARTLABEL="devinfo" PARTUUID="55a5c3dd-193b-177e-64d1-664e7b46788a"
/dev/sde62: PARTLABEL="dip" PARTUUID="99e65f2a-7751-c889-3aae-f7da066b659f"
/dev/sde63: PARTLABEL="apdp" PARTUUID="0a53e444-9b7d-7214-598d-17857de29af5"
/dev/sde64: PARTLABEL="msadp" PARTUUID="c668256b-2eba-a110-b03d-cba0a04c05ae"
/dev/sde65: PARTLABEL="spunvm" PARTUUID="0ed00786-8fe6-f114-a349-507766d26668"
/dev/sde66: PARTLABEL="splash" PARTUUID="1c445d09-286e-7b29-cf92-61c86ecbe66a"
/dev/sde67: PARTLABEL="limits" PARTUUID="5c1a5d8b-a10b-4b4e-29c5-c0477e5fae52"
/dev/sde68: PARTLABEL="toolsfv" PARTUUID="63893bdf-3141-3655-72ab-9c08e0089088"
/dev/sde69: SEC_TYPE="msdos" LABEL="LOGFS" UUID="D273-55EA" BLOCK_SIZE="4096" TYPE="vfat" PARTLABEL="logfs" PARTUUID="3fdeb6d4-674f-c5cd-3c4c-9ac8cc3f058b"
/dev/sde6: PARTLABEL="mdtpsecapp_a" PARTUUID="37f51290-80f6-60ae-a703-290353b5bd99"
/dev/sde70: PARTLABEL="sti" PARTUUID="857f6491-4f25-846d-0f4b-6b0ec6224272"
/dev/sde71: PARTLABEL="logdump" PARTUUID="5e20f984-aef5-f03c-04a3-aea4c71f6c18"
/dev/sde72: PARTLABEL="ImageFv" PARTUUID="cd1efda2-38b6-99bc-5a9d-e1e6320797ab"
/dev/sde7: PARTLABEL="mdtp_a" PARTUUID="65ff1e4c-fcd5-2b40-4633-f629b2af4f03"
/dev/sde8: PARTLABEL="abl_a" PARTUUID="cc85c2bd-e456-dea5-a0d1-b42da4c942d5"
/dev/sde9: LABEL="dsp" UUID="af32c008-2a39-7e5b-a5dc-201456d93103" BLOCK_SIZE="4096" TYPE="ext4" PARTLABEL="dsp_a" PARTUUID="59316eb5-a374-1a87-aec0-1f00d095b771"
/dev/sdf1: PARTLABEL="ALIGN_TO_128K_2" PARTUUID="24ba638c-a683-3c15-ecf4-6a4bafed73f2"
/dev/sdf2: PARTLABEL="modemst1" PARTUUID="3afee945-79eb-a5b5-0cd5-20414807b674"
/dev/sdf3: PARTLABEL="modemst2" PARTUUID="17ee895e-114e-b212-2bbe-0bb4fbdaa8a5"
/dev/sdf4: PARTLABEL="fsg" PARTUUID="cc233967-e1f4-7974-355b-44436976bf64"
/dev/sdf5: PARTLABEL="fsc" PARTUUID="81813b42-bc01-6ba0-2486-72dbb3c8d231"
/dev/zram0: LABEL="zram_swap" UUID="cb228574-bb8f-407c-9624-f913b7fa4733" TYPE="swap"
oneplus-enchilada:/home/user# blkid | sort -V
/dev/dm-0: LABEL="pmOS_boot" UUID="e577a746-23fc-4dfb-a855-c81e5ceede8b" BLOCK_SIZE="4096" TYPE="ext2"
/dev/dm-1: LABEL="pmOS_root" UUID="3c266d0a-d0b6-45fb-a367-d28056dec1d4" BLOCK_SIZE="4096" TYPE="ext4"
/dev/sda1: PARTLABEL="ssd" PARTUUID="90b16030-9d6e-8b41-f47f-575a2b601c11"
/dev/sda2: UUID="97df8ae2-99d3-45a6-a09e-8c7a1691247c" BLOCK_SIZE="4096" TYPE="ext4" PARTLABEL="persist" PARTUUID="8c782723-4bd3-5914-e3b7-90a2de933a04"
/dev/sda3: PARTLABEL="misc" PARTUUID="77e2d4ef-a9ee-4801-35c6-c9dfbee4ccda"
/dev/sda4: PARTLABEL="param" PARTUUID="162b5d9c-7558-db06-6f18-892bccae8cad"
/dev/sda5: PARTLABEL="keystore" PARTUUID="5478c133-c748-702a-e3d4-7b7a980b0da4"
/dev/sda6: PARTLABEL="frp" PARTUUID="c94d3031-3b43-5dd0-fc74-d45f925462a5"
/dev/sda7: LABEL="op2" UUID="05e2189f-0ff6-4219-a490-b6f4323cbc88" BLOCK_SIZE="4096" TYPE="ext4" PARTLABEL="op2" PARTUUID="2d46c473-838e-4d97-a824-f447f846075f"
/dev/sda8: PARTLABEL="oem_dycnvbk" PARTUUID="3be0090d-52d4-8b6b-5366-27079fc3b574"
/dev/sda9: PARTLABEL="oem_stanvbk" PARTUUID="bc19edc6-8498-60de-980d-269f4925fb7b"
/dev/sda10: PARTLABEL="reserve1" PARTUUID="698a851a-f33f-cd61-0d1b-826603a77ca4"
/dev/sda11: PARTLABEL="reserve2" PARTUUID="48ca4c46-75c1-3f5b-7c4a-846026908c6f"
/dev/sda12: PARTLABEL="config" PARTUUID="ed8f57c6-2bb7-4622-73ff-042e5dce6fb0"
/dev/sda13: LABEL="/" UUID="74ab2787-89b4-5f32-9f21-2c085107ac1e" BLOCK_SIZE="4096" TYPE="ext4" PARTLABEL="system_a" PARTUUID="19cc22c8-45f4-27de-fa05-9f48999ec5d3"
/dev/sda14: LABEL="/" UUID="c070087f-00b9-5294-8485-a9542d23a5c4" BLOCK_SIZE="4096" TYPE="ext4" PARTLABEL="system_b" PARTUUID="fb77c341-58db-78b5-3212-a3e8c80d2363"
/dev/sda15: LABEL="odm" UUID="1778e345-2c06-433d-84ca-5b0aabc12474" BLOCK_SIZE="4096" TYPE="ext4" PARTLABEL="odm_a" PARTUUID="08aec4a5-b82b-c2cf-dca7-fcca675360b2"
/dev/sda16: LABEL="odm" UUID="1778e345-2c06-433d-84ca-5b0aabc12474" BLOCK_SIZE="4096" TYPE="ext4" PARTLABEL="odm_b" PARTUUID="196a8d0f-c73a-078f-9c4a-620bf7c882b0"
/dev/sda17: PTUUID="d7b6d7b7" PTTYPE="dos" PARTLABEL="userdata" PARTUUID="d7201bd9-a5a4-7413-705e-42ddb9906ca8"
/dev/sdb1: PARTLABEL="xbl_a" PARTUUID="9dd5b108-3d9d-9df6-273a-3fdeabb0dd58"
/dev/sdb2: PARTLABEL="xbl_config_a" PARTUUID="0b8aa997-f093-80de-d0eb-b822f3a4bbaa"
/dev/sdc1: PARTLABEL="xbl_b" PARTUUID="9795e8c7-a4bc-37ee-ec57-f2404a97b407"
/dev/sdc2: PARTLABEL="xbl_config_b" PARTUUID="67af4168-1602-8b9f-3e4f-576106777ed3"
/dev/sdd1: PARTLABEL="ALIGN_TO_128K_1" PARTUUID="26ffa336-0b0d-07fc-8acb-a95637524e33"
/dev/sdd2: PARTLABEL="cdt" PARTUUID="a0ec5e99-cf0e-3337-b43c-59b5f04cc668"
/dev/sdd3: PARTLABEL="ddr" PARTUUID="b18ff3f4-0b09-bf50-f8a7-74f85d8c6cfd"
/dev/sde1: PARTLABEL="aop_a" PARTUUID="18a9eb6c-34ba-778a-378e-24a2ca53423d"
/dev/sde2: PARTLABEL="tz_a" PARTUUID="c468c1af-db1c-568f-8ec7-53b735416d25"
/dev/sde3: PARTLABEL="hyp_a" PARTUUID="ba8bafdb-7f50-1e8f-346a-c13abf300941"
/dev/sde4: SEC_TYPE="msdos" UUID="00BC-614E" BLOCK_SIZE="4096" TYPE="vfat" PARTLABEL="modem_a" PARTUUID="761d9a2e-73d1-bf89-52dd-f16d65f3949f"
/dev/sde5: SEC_TYPE="msdos" UUID="00BC-614E" BLOCK_SIZE="4096" TYPE="vfat" PARTLABEL="bluetooth_a" PARTUUID="690f015d-1f6a-9041-7a71-41457afcc4c3"
/dev/sde6: PARTLABEL="mdtpsecapp_a" PARTUUID="37f51290-80f6-60ae-a703-290353b5bd99"
/dev/sde7: PARTLABEL="mdtp_a" PARTUUID="65ff1e4c-fcd5-2b40-4633-f629b2af4f03"
/dev/sde8: PARTLABEL="abl_a" PARTUUID="cc85c2bd-e456-dea5-a0d1-b42da4c942d5"
/dev/sde9: LABEL="dsp" UUID="af32c008-2a39-7e5b-a5dc-201456d93103" BLOCK_SIZE="4096" TYPE="ext4" PARTLABEL="dsp_a" PARTUUID="59316eb5-a374-1a87-aec0-1f00d095b771"
/dev/sde10: PARTLABEL="keymaster_a" PARTUUID="c15b5503-05e2-74ea-9dd1-3ddaa1933664"
/dev/sde11: PARTLABEL="boot_a" PARTUUID="1dd3a986-997c-0c48-1d1b-b0d0399f3153"
/dev/sde12: PARTLABEL="cmnlib_a" PARTUUID="4be9c484-fead-b4df-028b-c42f19bb771d"
/dev/sde13: PARTLABEL="cmnlib64_a" PARTUUID="7a676289-f499-4849-2e3e-6110c72f8b3c"
/dev/sde14: PARTLABEL="devcfg_a" PARTUUID="cb0d0e64-6ccf-412c-a1a1-6a6f6899c0cf"
/dev/sde15: PARTLABEL="qupfw_a" PARTUUID="0c29a859-40c6-0351-85f0-40bcb67a9e70"
/dev/sde16: LABEL="vendor" UUID="0c70056c-5d72-53f7-93e4-c56a1b61e40a" BLOCK_SIZE="4096" TYPE="ext4" PARTLABEL="vendor_a" PARTUUID="b5e4e456-0c89-c7aa-4671-dfcfa58c4a50"
/dev/sde17: PARTLABEL="vbmeta_a" PARTUUID="ea36dbc9-3ef3-8033-289a-8fd65ad70cd8"
/dev/sde18: PARTLABEL="dtbo_a" PARTUUID="88acca40-bf3f-05c5-c64c-57d0b94ef345"
/dev/sde19: PARTLABEL="storsec_a" PARTUUID="e2d23828-7af0-2a38-9e54-dc992d9ec299"
/dev/sde20: PARTLABEL="LOGO_a" PARTUUID="7d09acd3-fdc7-24a3-3ef1-7231c8557dc5"
/dev/sde21: PARTLABEL="fw_4j1ed_a" PARTUUID="7984bbfb-9c60-969a-511b-1927110bd37b"
/dev/sde22: PARTLABEL="fw_4u1ea_a" PARTUUID="0de21a2d-16ed-a2a5-3628-27cbf5f6a7b6"
/dev/sde23: PARTLABEL="fw_ufs3_a" PARTUUID="99cfddd1-c334-fbd3-4592-dd3dae42d7e4"
/dev/sde24: PARTLABEL="fw_ufs4_a" PARTUUID="79c101ca-486a-4761-d1fd-42d4a90ecd8f"
/dev/sde25: PARTLABEL="fw_ufs5_a" PARTUUID="ba5b3351-d3c7-334a-5d4f-6bc6ee18c76e"
/dev/sde26: PARTLABEL="fw_ufs6_a" PARTUUID="959d46fc-62d6-1b81-a3c7-4782878bf939"
/dev/sde27: PARTLABEL="fw_ufs7_a" PARTUUID="8aef2687-45eb-6104-822c-ba3ebcda03aa"
/dev/sde28: PARTLABEL="fw_ufs8_a" PARTUUID="7dca79b5-2813-87dd-7241-2538fa3f6809"
/dev/sde29: PARTLABEL="aop_b" PARTUUID="ef572bf2-e170-61e2-636d-1e00d9bdb9ad"
/dev/sde30: PARTLABEL="tz_b" PARTUUID="8eec337d-8ffd-bcef-45dd-fbbaef420af8"
/dev/sde31: PARTLABEL="hyp_b" PARTUUID="c2f15698-daec-9aa2-a097-f7c0ec244fa1"
/dev/sde32: SEC_TYPE="msdos" UUID="00BC-614E" BLOCK_SIZE="4096" TYPE="vfat" PARTLABEL="modem_b" PARTUUID="d80fff11-faf5-e403-1e07-b3f92da5df14"
/dev/sde33: SEC_TYPE="msdos" UUID="00BC-614E" BLOCK_SIZE="4096" TYPE="vfat" PARTLABEL="bluetooth_b" PARTUUID="6c8f5520-3b25-f1aa-113e-5444ec6b7bc3"
/dev/sde34: PARTLABEL="mdtpsecapp_b" PARTUUID="c093098a-c29c-67d5-87cc-b823325f500c"
/dev/sde35: PARTLABEL="mdtp_b" PARTUUID="5d694b4a-8561-d7fd-4f88-192d902bf8d1"
/dev/sde36: PARTLABEL="abl_b" PARTUUID="f83f5a71-7460-d50f-af91-e30535309ccb"
/dev/sde37: LABEL="dsp" UUID="af32c008-2a39-7e5b-a5dc-201456d93103" BLOCK_SIZE="4096" TYPE="ext4" PARTLABEL="dsp_b" PARTUUID="79ba0936-0c74-64ae-e995-8c041e2d684b"
/dev/sde38: PARTLABEL="keymaster_b" PARTUUID="ede27ba9-4c06-b49f-e30e-cd081c12cefa"
/dev/sde39: PARTLABEL="boot_b" PARTUUID="45105095-3847-4657-51f2-2a0144550453"
/dev/sde40: PARTLABEL="cmnlib_b" PARTUUID="cdb50a31-5cb3-646a-5ffa-37f1fc7cf02d"
/dev/sde41: PARTLABEL="cmnlib64_b" PARTUUID="6f4fa8ad-0733-6d2a-4641-3335db470550"
/dev/sde42: PARTLABEL="devcfg_b" PARTUUID="d501d92e-854e-6cdf-bd8a-b29c5303b24f"
/dev/sde43: PARTLABEL="qupfw_b" PARTUUID="209724bc-ee5d-f227-2c4d-a3e8e9364db1"
/dev/sde44: PARTLABEL="vendor_b" PARTUUID="dbe58f5c-303e-d453-ef76-91c9c88cbd7e"
/dev/sde45: PARTLABEL="vbmeta_b" PARTUUID="f0c2dc1c-9b74-9dc4-b0da-4887611bb8b7"
/dev/sde46: PARTLABEL="dtbo_b" PARTUUID="a82e816e-fe23-6ac6-1015-ae0344a28b28"
/dev/sde47: PARTLABEL="storsec_b" PARTUUID="9538581c-f700-75cd-d11c-ab5d4301790e"
/dev/sde48: PARTLABEL="LOGO_b" PARTUUID="262aaa7b-61c0-b7f2-9526-4ce1e664e96f"
/dev/sde49: PARTLABEL="fw_4j1ed_b" PARTUUID="76de293a-76e8-f885-d33c-c9caa5f95104"
/dev/sde50: PARTLABEL="fw_4u1ea_b" PARTUUID="164fd6d9-9ca4-478b-1703-5a764b214c3c"
/dev/sde51: PARTLABEL="fw_ufs3_b" PARTUUID="9f5d63d8-77bd-dbc0-61f2-fa2e0b338a03"
/dev/sde52: PARTLABEL="fw_ufs4_b" PARTUUID="251cb7ad-053c-fa41-115f-767116727e4e"
/dev/sde53: PARTLABEL="fw_ufs5_b" PARTUUID="6dc25d07-16a8-0e5f-eca6-e6bf3881195d"
/dev/sde54: PARTLABEL="fw_ufs6_b" PARTUUID="b10083d8-de96-7a82-f72a-12baaa0d3751"
/dev/sde55: PARTLABEL="fw_ufs7_b" PARTUUID="9dd01a5a-d0d0-572a-6b93-e93c1c7c65fc"
/dev/sde56: PARTLABEL="fw_ufs8_b" PARTUUID="630122ae-5505-88cb-f65b-0e694810a868"
/dev/sde57: PARTLABEL="minidump" PARTUUID="b5df8fd2-39ab-e2b1-0123-0d6d07948f31"
/dev/sde58: PARTLABEL="boot_aging" PARTUUID="b3d11f25-855a-524f-b3af-961ad2a937ab"
/dev/sde59: LABEL="op1" UUID="58a6b23e-5ce2-4d69-881b-69d3b061564b" BLOCK_SIZE="4096" TYPE="ext4" PARTLABEL="op1" PARTUUID="2190addd-4b30-9c0e-7b43-44e1c943850f"
/dev/sde60: PARTLABEL="sec" PARTUUID="40b72c38-8636-49d4-7c38-dd1274ab928b"
/dev/sde61: PARTLABEL="devinfo" PARTUUID="55a5c3dd-193b-177e-64d1-664e7b46788a"
/dev/sde62: PARTLABEL="dip" PARTUUID="99e65f2a-7751-c889-3aae-f7da066b659f"
/dev/sde63: PARTLABEL="apdp" PARTUUID="0a53e444-9b7d-7214-598d-17857de29af5"
/dev/sde64: PARTLABEL="msadp" PARTUUID="c668256b-2eba-a110-b03d-cba0a04c05ae"
/dev/sde65: PARTLABEL="spunvm" PARTUUID="0ed00786-8fe6-f114-a349-507766d26668"
/dev/sde66: PARTLABEL="splash" PARTUUID="1c445d09-286e-7b29-cf92-61c86ecbe66a"
/dev/sde67: PARTLABEL="limits" PARTUUID="5c1a5d8b-a10b-4b4e-29c5-c0477e5fae52"
/dev/sde68: PARTLABEL="toolsfv" PARTUUID="63893bdf-3141-3655-72ab-9c08e0089088"
/dev/sde69: SEC_TYPE="msdos" LABEL="LOGFS" UUID="D273-55EA" BLOCK_SIZE="4096" TYPE="vfat" PARTLABEL="logfs" PARTUUID="3fdeb6d4-674f-c5cd-3c4c-9ac8cc3f058b"
/dev/sde70: PARTLABEL="sti" PARTUUID="857f6491-4f25-846d-0f4b-6b0ec6224272"
/dev/sde71: PARTLABEL="logdump" PARTUUID="5e20f984-aef5-f03c-04a3-aea4c71f6c18"
/dev/sde72: PARTLABEL="ImageFv" PARTUUID="cd1efda2-38b6-99bc-5a9d-e1e6320797ab"
/dev/sdf1: PARTLABEL="ALIGN_TO_128K_2" PARTUUID="24ba638c-a683-3c15-ecf4-6a4bafed73f2"
/dev/sdf2: PARTLABEL="modemst1" PARTUUID="3afee945-79eb-a5b5-0cd5-20414807b674"
/dev/sdf3: PARTLABEL="modemst2" PARTUUID="17ee895e-114e-b212-2bbe-0bb4fbdaa8a5"
/dev/sdf4: PARTLABEL="fsg" PARTUUID="cc233967-e1f4-7974-355b-44436976bf64"
/dev/sdf5: PARTLABEL="fsc" PARTUUID="81813b42-bc01-6ba0-2486-72dbb3c8d231"
/dev/zram0: LABEL="zram_swap" UUID="cb228574-bb8f-407c-9624-f913b7fa4733" TYPE="swap"
```