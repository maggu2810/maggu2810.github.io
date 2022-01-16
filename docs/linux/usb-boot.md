# USB Multiboot

This side describes how to create a USB stick that can boot multiple operating systems. Additionally it should work for (U)EFI and legacy BIOS systems.

First let's state: It works.

But there could be a problem supporting legacy BIOS systems and use big size USB sticks. So, it is not really a problem, but you need to consider it by the design of the partition layout. To summarize it shortly: The data that could be accessed by legacy BIOS on USB stick is limited by their size. I run into the problem that the BIOS of some of my machines cannot access data that is stored above the 128 GiB / 137 GB limit.

Some further details could be found here:

* [History of BIOS and IDE limits](https://tldp.org/HOWTO/Large-Disk-HOWTO-4.html)
* [Legacy BIOS Access Range Limitation](https://ventoy.net/en/doc_legacy_limit.html)
* [The 128Gib/137GB USB BIOS bug!](https://www.easy2boot.com/faq-/a137gb-bios-bug/)

This could be worked around to place all files that needs to be accessed on bootup initially are places below the 128 GiB limit. For Linux systems this would be the kernel and the initramfs / initrd. All other files from root filesystem will be accessed by the kernel driver and so the limit does not exist anymore.

Personally I would consider what you want to achieve. If you want to create an USB stick with multiple Linux systems using separate boot partitions could increase the initial setup. But it is all doable without big headache.

As written in the first link above there could be other limits too, if you use very old machines, but I did not run into yet.

## Additional lecture

* [UEFI (Debian Wiki)](https://wiki.debian.org/UEFI)
* [Partition Alignment detailed explanation](https://www.thomas-krenn.com/en/wiki/Partition_Alignment_detailed_explanation)
* [Ein Notfallsystem auf einem USB-Stick installieren](https://wiki.debianforum.de/Ein_Notfallsystem_auf_einem_USB-Stick_installieren)
* [Managing EFI Boot Loaders for Linux](https://www.rodsbooks.com/efi-bootloaders/index.html)
* [GRUB, disable chain of trust](https://wejn.org/2021/09/fixing-grub-verification-requested-nobody-cares/)


For further information about the BIOS Boot Partition have a look at [BIOS boot partition (Wikipedia en)](https://en.wikipedia.org/wiki/BIOS_boot_partition) especially on the image.

## Example partition layouts

### Work around 127 GiB limit

| Partition | Use Case | Size | Partition Type | Name |
| ---- | ---- | ---- | ---- | ---- |
| `/dev/sdb1` | EFI System Partition | +2G | ef00 | EFI_UMB |
| `/dev/sdb2` | BIOS Boot Partition | +100M | ef02 | BIOSBOOT_UMB |
| `/dev/sdb3` | boot directory of all linux systems | +8G | 8300 | LINBOOT_UMB |
| `/dev/sdb4` | Windows Image 1 | +10G | 0700 | WIN1_UMB |
| `/dev/sdb5` | Windows Image 2 | +10G | 0700 | WIN2_UMB |
| `/dev/sdb6` | remaining stuff of all linux systems | remaining | 8300 | LIN_UMB |

### No special workaround, Linux only

| Partition | Use Case | Size | Partition Type | Name |
| ---- | ---- | ---- | ---- | ---- |
| `/dev/sdb1` | EFI System Partition | +2G | ef00 | EFI_USBLIN |
| `/dev/sdb2` | BIOS Boot Partition | +100M | ef02 | BIOSBOOT_USBLIN |
| `/dev/sdb3` | Linux etc. | remaining | 8300 | BTRFS_USBLIN |

## Example partitioning + format

### No special BIOS workaround

```
# as root
gdisk /dev/sdb
```

```sh
GPT fdisk (gdisk) version 1.0.8

Partition table scan:
  MBR: protective
  BSD: not present
  APM: not present
  GPT: present

Found valid GPT with protective MBR; using GPT.

Command (? for help): o
This option deletes all partitions and creates a new protective MBR.
Proceed? (Y/N): Y

Command (? for help): n
Partition number (1-128, default 1): 
First sector (34-500170718, default = 2048) or {+-}size{KMGTP}: 
Last sector (2048-500170718, default = 500170718) or {+-}size{KMGTP}: +2G
Current type is 8300 (Linux filesystem)
Hex code or GUID (L to show codes, Enter = 8300): ef00
Changed type of partition to 'EFI system partition'

Command (? for help): n
Partition number (2-128, default 2): 
First sector (34-500170718, default = 4196352) or {+-}size{KMGTP}: 
Last sector (4196352-500170718, default = 500170718) or {+-}size{KMGTP}: +100M
Current type is 8300 (Linux filesystem)
Hex code or GUID (L to show codes, Enter = 8300): ef02
Changed type of partition to 'BIOS boot partition'

Command (? for help): n
Partition number (3-128, default 3): 
First sector (34-500170718, default = 4401152) or {+-}size{KMGTP}: 
Last sector (4401152-500170718, default = 500170718) or {+-}size{KMGTP}: 
Current type is 8300 (Linux filesystem)
Hex code or GUID (L to show codes, Enter = 8300): 8300
Changed type of partition to 'Linux filesystem'

Command (? for help): c
Partition number (1-3): 1
Enter name: EFI_USBLIN

Command (? for help): c
Partition number (1-3): 2
Enter name: BIOSBOOT_USBLIN

Command (? for help): c
Partition number (1-3): 3
Enter name: BTRFS_USBLIN

Command (? for help): wq	

Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING
PARTITIONS!!

Do you want to proceed? (Y/N): Y
OK; writing new GUID partition table (GPT) to /dev/sdb.
The operation has completed successfully.
```

Unplug and plug in the USB stick.

```
mkfs.vfat -n EFI_USBLIN /dev/sdb1
mkfs.btrfs -L BTRFS_USBLIN /dev/sdb3
```

```
blkid /dev/sdb*
```

```
/dev/sdb: PTUUID="832d97be-48bd-4c42-866e-929e88b2ee90" PTTYPE="gpt"
/dev/sdb1: LABEL_FATBOOT="EFI_USBLIN" LABEL="EFI_USBLIN" UUID="6641-AD18" BLOCK_SIZE="512" TYPE="vfat" PARTLABEL="EFI_USBLIN" PARTUUID="ca5a6790-1e89-4ef3-a5e9-cb8c18aa96f0"
/dev/sdb2: PARTLABEL="BIOSBOOT_USBLIN" PARTUUID="946032eb-4033-474a-8a64-0fcdef93b510"
/dev/sdb3: LABEL="BTRFS_USBLIN" UUID="c40a464a-f21a-48af-b655-b1e75571788f" UUID_SUB="2c90051d-92cf-468d-ba3d-fe784e216bdb" BLOCK_SIZE="4096" TYPE="btrfs" PARTLABEL="BTRFS_USBLIN" PARTUUID="14062ad2-9bf1-4d2e-b33f-c4962961d813"
```

### No special BIOS workaround, EFI only, swap

```sh
gdisk /dev/sdb
```

```text
GPT fdisk (gdisk) version 1.0.8

Partition table scan:
  MBR: MBR only
  BSD: not present
  APM: not present
  GPT: not present


***************************************************************
Found invalid GPT and valid MBR; converting MBR to GPT format
in memory. THIS OPERATION IS POTENTIALLY DESTRUCTIVE! Exit by
typing 'q' if you don't want to convert your MBR partitions
to GPT format!
***************************************************************


Warning! Secondary partition table overlaps the last partition by
33 blocks!
You will need to delete this partition or resize it in another utility.

Command (? for help): o
This option deletes all partitions and creates a new protective MBR.
Proceed? (Y/N): Y

Command (? for help): n
Partition number (1-128, default 1): 
First sector (34-500170718, default = 2048) or {+-}size{KMGTP}: 
Last sector (2048-500170718, default = 500170718) or {+-}size{KMGTP}: +2G
Current type is 8300 (Linux filesystem)
Hex code or GUID (L to show codes, Enter = 8300): ef00
Changed type of partition to 'EFI system partition'

Command (? for help): n
Partition number (2-128, default 2): 
First sector (34-500170718, default = 4196352) or {+-}size{KMGTP}: 
Last sector (4196352-500170718, default = 500170718) or {+-}size{KMGTP}: +32G
Current type is 8300 (Linux filesystem)
Hex code or GUID (L to show codes, Enter = 8300): 8200
Changed type of partition to 'Linux swap'

Command (? for help): n
Partition number (3-128, default 3): 
First sector (34-500170718, default = 71305216) or {+-}size{KMGTP}: 
Last sector (71305216-500170718, default = 500170718) or {+-}size{KMGTP}: 
Current type is 8300 (Linux filesystem)
Hex code or GUID (L to show codes, Enter = 8300): 
Changed type of partition to 'Linux filesystem'

Command (? for help): c
Partition number (1-3): 1
Enter name: EFI_USBLIN

Command (? for help): c
Partition number (1-3): 2
Enter name: SWAP_USBLIN

Command (? for help): c
Partition number (1-3): 3
Enter name: BTRFS_USBLIN

Command (? for help): wq

Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING
PARTITIONS!!

Do you want to proceed? (Y/N): Y
OK; writing new GUID partition table (GPT) to /dev/sdb.
The operation has completed successfully.
```

```sh
mkfs.vfat -n EFI_USBLIN /dev/sdb1
mkswap -L SWAP_USBLIN /dev/sdb2
mkfs.btrfs -L BTRFS_USBLIN /dev/sdb3
```

```sh
blkid /dev/sdb*
```

```text
/dev/sdb: PTUUID="192efcf3-550f-48b3-b997-3f6298eb9900" PTTYPE="gpt"
/dev/sdb1: LABEL_FATBOOT="EFI_USBLIN" LABEL="EFI_USBLIN" UUID="BD6C-E793" BLOCK_SIZE="512" TYPE="vfat" PARTLABEL="EFI_USBLIN" PARTUUID="208d55b6-112c-4bd9-a0fd-5cb309b72fd7"
/dev/sdb2: LABEL="SWAP_USBLIN" UUID="7ee2e029-181d-4fe6-b90f-ffcfc0a19640" TYPE="swap" PARTLABEL="SWAP_USBLIN" PARTUUID="e5f81256-52fd-4bc9-a0e6-ae22666753e3"
/dev/sdb3: LABEL="BTRFS_USBLIN" UUID="0f7a4702-0202-4e56-b3ba-e03c180ea621" UUID_SUB="c01dfff8-89f1-4d04-b3a7-0e304e93f2d7" BLOCK_SIZE="4096" TYPE="btrfs" PARTLABEL="BTRFS_USBLIN" PARTUUID="47ba980c-87d0-4e6c-9214-7a51f036a62d"
```

```bash
export MP_BTRFS_USBLIN=$(mktemp -d)
mount /dev/disk/by-label/BTRFS_USBLIN "${MP_BTRFS_USBLIN}"
btrfs subvolume create "${MP_BTRFS_USBLIN}"/ubuntu.focal
btrfs subvolume create "${MP_BTRFS_USBLIN}"/archlinux
umount "${MP_BTRFS_USBLIN}"
rmdir "${MP_BTRFS_USBLIN}"
unset MP_BTRFS_USBLIN
```

```bash
mkdir -p /mnt/usblin_ubuntu_focal
mount /dev/disk/by-label/BTRFS_USBLIN -o subvol=ubuntu.focal /mnt/usblin_ubuntu_focal
mkdir -p /mnt/usblin_archlinux
mount /dev/disk/by-label/BTRFS_USBLIN -o subvol=archlinux /mnt/usblin_archlinux

# BEG: bootstrap - simple
debootstrap --arch=amd64 focal /mnt/usblin_ubuntu_focal
tar xzf /tmp/archlinux-bootstrap-2022.01.01-x86_64.tar.gz --numeric-owner --strip-components=1 -C /mnt/usblin_archlinux/
# END: bootstrap - simple

# TODO: setup bootstrapped simple systems (see links)

umount /mnt/usblin_archlinux
umount /mnt/usblin_ubuntu_focal
```

## Bootstrap a Linux System

Bootstrap a Linux System to a subvolume of "BTRFS_USBLIN".

```
mkdir /tmp/3; mount /dev/sdb3 /tmp/3
btrfs subvolume create /tmp/3/ubuntu.focal
debootstrap --arch=amd64 focal /tmp/3/ubuntu.focal/
```

For further instructions see e.g. [bootstrap](bootstrap.md)

```
umount /tmp/3
```

## Bootloader

### Grub (multiboot, arch linux loader)

We decide to use Grub as we can use one configuration for:

* boot from PCs with BIOS or UEFI Legacy Mode
* boot from 64-bit UEFI
* boot from 32-bit UEFI

We apply the steps in an Arch Linux system as we can install ONE grub for all targets. On Debian / Ubuntu you need to install different packages (and remove them again, as only one can be installed in parallel).

So, let's assume you have a running Arch Linux system (or chroot setup with all relevant dev, proc, sys etc. stuff).

Install Grub

```
pacman -S grub
```

Mount ESP

```
mkdir -p /efi
mount /dev/disk/by-label/EFI_USBLIN /efi
```

```
mkdir -p /efi/EFI/BOOT/
grub-install --target=i386-pc --boot-directory=/efi/boot /dev/sdb

grub-install --target=i386-efi --boot-directory=/efi/boot --efi-directory=/efi --bootloader-id=grub --no-nvram
mkdir -p /efi/EFI/BOOT/
cp /efi/EFI/grub/grubia32.efi /efi/EFI/BOOT/BOOTIA32.EFI

grub-install --target=x86_64-efi --boot-directory=/efi/boot --efi-directory=/efi --bootloader-id=grub --no-nvram
mkdir -p /efi/EFI/BOOT/
cp /efi/EFI/grub/grubx64.efi /efi/EFI/BOOT/BOOTX64.EFI
```

Now edit the configuration file `/efi/boot/grub/grub.cfg`

```
search.fs_uuid 6641-AD18 root hd1,gpt1 
set prefix=($root)'/boot/grub'
configfile $prefix/grub-manual.cfg
```

and file `/efi/boot/grub/grub-manual.cfg`:

```
insmod part_gpt
insmod btrfs
insmod gzio
insmod gettext
insmod all_video
insmod gfxterm

set menu_color_normal=white/black
set menu_color_highlight=black/light-gray

set default=0



set MPUUID="c40a464a-f21a-48af-b655-b1e75571788f"
search --no-floppy --fs-uuid --set=root $MPUUID


menuentry 'Ubuntu Focal' {
        set MPSUBVOL="ubuntu.focal"
        linux  /$MPSUBVOL/boot/vmlinuz rootwait root=UUID=$MPUUID rootflags=subvol=$MPSUBVOL rw
        initrd  /$MPSUBVOL/boot/initrd.img
}

menuentry 'Arch Linux' {
        set MPSUBVOL="archlinux"
        linux  /$MPSUBVOL/boot/vmlinuz-linux rootwait root=UUID=$MPUUID rootflags=subvol=$MPSUBVOL rw
        initrd  /$MPSUBVOL/boot/initramfs-linux.img
}
```

### grub (EFI x64, ubuntu signed loader)

```sh
export MPBS="/mnt/usblin_ubuntu_focal"

mount /dev/disk/by-label/BTRFS_USBLIN -o subvol=ubuntu.focal /mnt/usblin_ubuntu_focal

mount -t proc /proc "${MPBS}/proc"
mount --rbind /sys "${MPBS}/sys"
mount --rbind /run "${MPBS}/run"
mount --rbind /dev "${MPBS}/dev"

chroot "${MPBS}"

# BEG: inside chroot

apt install shim-signed grub-efi-amd64-signed

mkdir -p /efi
mount /dev/disk/by-label/EFI_USBLIN /efi

grub-install --uefi-secure-boot --target=x86_64-efi --boot-directory=/efi/boot --efi-directory=/efi --bootloader-id=grub --no-nvram

cd /efi
find | sort
```

```text
.
./boot
./boot/grub
./boot/grub/fonts
./boot/grub/fonts/unicode.pf2
./boot/grub/grubenv
./boot/grub/x86_64-efi
./boot/grub/x86_64-efi/acpi.mod
./boot/grub/x86_64-efi/adler32.mod
./boot/grub/x86_64-efi/affs.mod
./boot/grub/x86_64-efi/afs.mod
./boot/grub/x86_64-efi/ahci.mod
./boot/grub/x86_64-efi/all_video.mod
./boot/grub/x86_64-efi/aout.mod
./boot/grub/x86_64-efi/appleldr.mod
./boot/grub/x86_64-efi/archelp.mod
./boot/grub/x86_64-efi/ata.mod
./boot/grub/x86_64-efi/at_keyboard.mod
./boot/grub/x86_64-efi/backtrace.mod
./boot/grub/x86_64-efi/bfs.mod
./boot/grub/x86_64-efi/bitmap.mod
./boot/grub/x86_64-efi/bitmap_scale.mod
./boot/grub/x86_64-efi/blocklist.mod
./boot/grub/x86_64-efi/boot.mod
./boot/grub/x86_64-efi/bsd.mod
./boot/grub/x86_64-efi/bswap_test.mod
./boot/grub/x86_64-efi/btrfs.mod
./boot/grub/x86_64-efi/bufio.mod
./boot/grub/x86_64-efi/cat.mod
./boot/grub/x86_64-efi/cbfs.mod
./boot/grub/x86_64-efi/cbls.mod
./boot/grub/x86_64-efi/cbmemc.mod
./boot/grub/x86_64-efi/cbtable.mod
./boot/grub/x86_64-efi/cbtime.mod
./boot/grub/x86_64-efi/chain.mod
./boot/grub/x86_64-efi/cmdline_cat_test.mod
./boot/grub/x86_64-efi/cmp.mod
./boot/grub/x86_64-efi/cmp_test.mod
./boot/grub/x86_64-efi/command.lst
./boot/grub/x86_64-efi/configfile.mod
./boot/grub/x86_64-efi/core.efi
./boot/grub/x86_64-efi/cpio_be.mod
./boot/grub/x86_64-efi/cpio.mod
./boot/grub/x86_64-efi/cpuid.mod
./boot/grub/x86_64-efi/crc64.mod
./boot/grub/x86_64-efi/cryptodisk.mod
./boot/grub/x86_64-efi/crypto.lst
./boot/grub/x86_64-efi/crypto.mod
./boot/grub/x86_64-efi/cs5536.mod
./boot/grub/x86_64-efi/ctz_test.mod
./boot/grub/x86_64-efi/datehook.mod
./boot/grub/x86_64-efi/date.mod
./boot/grub/x86_64-efi/datetime.mod
./boot/grub/x86_64-efi/diskfilter.mod
./boot/grub/x86_64-efi/disk.mod
./boot/grub/x86_64-efi/div.mod
./boot/grub/x86_64-efi/div_test.mod
./boot/grub/x86_64-efi/dm_nv.mod
./boot/grub/x86_64-efi/echo.mod
./boot/grub/x86_64-efi/efifwsetup.mod
./boot/grub/x86_64-efi/efi_gop.mod
./boot/grub/x86_64-efi/efinet.mod
./boot/grub/x86_64-efi/efi_uga.mod
./boot/grub/x86_64-efi/ehci.mod
./boot/grub/x86_64-efi/elf.mod
./boot/grub/x86_64-efi/eval.mod
./boot/grub/x86_64-efi/exfat.mod
./boot/grub/x86_64-efi/exfctest.mod
./boot/grub/x86_64-efi/ext2.mod
./boot/grub/x86_64-efi/extcmd.mod
./boot/grub/x86_64-efi/f2fs.mod
./boot/grub/x86_64-efi/fat.mod
./boot/grub/x86_64-efi/file.mod
./boot/grub/x86_64-efi/fixvideo.mod
./boot/grub/x86_64-efi/font.mod
./boot/grub/x86_64-efi/fshelp.mod
./boot/grub/x86_64-efi/fs.lst
./boot/grub/x86_64-efi/functional_test.mod
./boot/grub/x86_64-efi/gcry_arcfour.mod
./boot/grub/x86_64-efi/gcry_blowfish.mod
./boot/grub/x86_64-efi/gcry_camellia.mod
./boot/grub/x86_64-efi/gcry_cast5.mod
./boot/grub/x86_64-efi/gcry_crc.mod
./boot/grub/x86_64-efi/gcry_des.mod
./boot/grub/x86_64-efi/gcry_dsa.mod
./boot/grub/x86_64-efi/gcry_idea.mod
./boot/grub/x86_64-efi/gcry_md4.mod
./boot/grub/x86_64-efi/gcry_md5.mod
./boot/grub/x86_64-efi/gcry_rfc2268.mod
./boot/grub/x86_64-efi/gcry_rijndael.mod
./boot/grub/x86_64-efi/gcry_rmd160.mod
./boot/grub/x86_64-efi/gcry_rsa.mod
./boot/grub/x86_64-efi/gcry_seed.mod
./boot/grub/x86_64-efi/gcry_serpent.mod
./boot/grub/x86_64-efi/gcry_sha1.mod
./boot/grub/x86_64-efi/gcry_sha256.mod
./boot/grub/x86_64-efi/gcry_sha512.mod
./boot/grub/x86_64-efi/gcry_tiger.mod
./boot/grub/x86_64-efi/gcry_twofish.mod
./boot/grub/x86_64-efi/gcry_whirlpool.mod
./boot/grub/x86_64-efi/geli.mod
./boot/grub/x86_64-efi/gettext.mod
./boot/grub/x86_64-efi/gfxmenu.mod
./boot/grub/x86_64-efi/gfxterm_background.mod
./boot/grub/x86_64-efi/gfxterm_menu.mod
./boot/grub/x86_64-efi/gfxterm.mod
./boot/grub/x86_64-efi/gptsync.mod
./boot/grub/x86_64-efi/grub.efi
./boot/grub/x86_64-efi/gzio.mod
./boot/grub/x86_64-efi/halt.mod
./boot/grub/x86_64-efi/hashsum.mod
./boot/grub/x86_64-efi/hdparm.mod
./boot/grub/x86_64-efi/hello.mod
./boot/grub/x86_64-efi/help.mod
./boot/grub/x86_64-efi/hexdump.mod
./boot/grub/x86_64-efi/hfs.mod
./boot/grub/x86_64-efi/hfspluscomp.mod
./boot/grub/x86_64-efi/hfsplus.mod
./boot/grub/x86_64-efi/http.mod
./boot/grub/x86_64-efi/iorw.mod
./boot/grub/x86_64-efi/iso9660.mod
./boot/grub/x86_64-efi/jfs.mod
./boot/grub/x86_64-efi/jpeg.mod
./boot/grub/x86_64-efi/keylayouts.mod
./boot/grub/x86_64-efi/keystatus.mod
./boot/grub/x86_64-efi/ldm.mod
./boot/grub/x86_64-efi/legacycfg.mod
./boot/grub/x86_64-efi/legacy_password_test.mod
./boot/grub/x86_64-efi/linux16.mod
./boot/grub/x86_64-efi/linuxefi.mod
./boot/grub/x86_64-efi/linux.mod
./boot/grub/x86_64-efi/loadbios.mod
./boot/grub/x86_64-efi/load.cfg
./boot/grub/x86_64-efi/loadenv.mod
./boot/grub/x86_64-efi/loopback.mod
./boot/grub/x86_64-efi/lsacpi.mod
./boot/grub/x86_64-efi/lsefimmap.mod
./boot/grub/x86_64-efi/lsefi.mod
./boot/grub/x86_64-efi/lsefisystab.mod
./boot/grub/x86_64-efi/lsmmap.mod
./boot/grub/x86_64-efi/ls.mod
./boot/grub/x86_64-efi/lspci.mod
./boot/grub/x86_64-efi/lssal.mod
./boot/grub/x86_64-efi/luks.mod
./boot/grub/x86_64-efi/lvm.mod
./boot/grub/x86_64-efi/lzopio.mod
./boot/grub/x86_64-efi/macbless.mod
./boot/grub/x86_64-efi/macho.mod
./boot/grub/x86_64-efi/mdraid09_be.mod
./boot/grub/x86_64-efi/mdraid09.mod
./boot/grub/x86_64-efi/mdraid1x.mod
./boot/grub/x86_64-efi/memdisk.mod
./boot/grub/x86_64-efi/memrw.mod
./boot/grub/x86_64-efi/minicmd.mod
./boot/grub/x86_64-efi/minix2_be.mod
./boot/grub/x86_64-efi/minix2.mod
./boot/grub/x86_64-efi/minix3_be.mod
./boot/grub/x86_64-efi/minix3.mod
./boot/grub/x86_64-efi/minix_be.mod
./boot/grub/x86_64-efi/minix.mod
./boot/grub/x86_64-efi/mmap.mod
./boot/grub/x86_64-efi/moddep.lst
./boot/grub/x86_64-efi/modinfo.sh
./boot/grub/x86_64-efi/morse.mod
./boot/grub/x86_64-efi/mpi.mod
./boot/grub/x86_64-efi/msdospart.mod
./boot/grub/x86_64-efi/mul_test.mod
./boot/grub/x86_64-efi/multiboot2.mod
./boot/grub/x86_64-efi/multiboot.mod
./boot/grub/x86_64-efi/nativedisk.mod
./boot/grub/x86_64-efi/net.mod
./boot/grub/x86_64-efi/newc.mod
./boot/grub/x86_64-efi/nilfs2.mod
./boot/grub/x86_64-efi/normal.mod
./boot/grub/x86_64-efi/ntfscomp.mod
./boot/grub/x86_64-efi/ntfs.mod
./boot/grub/x86_64-efi/odc.mod
./boot/grub/x86_64-efi/offsetio.mod
./boot/grub/x86_64-efi/ohci.mod
./boot/grub/x86_64-efi/part_acorn.mod
./boot/grub/x86_64-efi/part_amiga.mod
./boot/grub/x86_64-efi/part_apple.mod
./boot/grub/x86_64-efi/part_bsd.mod
./boot/grub/x86_64-efi/part_dfly.mod
./boot/grub/x86_64-efi/part_dvh.mod
./boot/grub/x86_64-efi/part_gpt.mod
./boot/grub/x86_64-efi/partmap.lst
./boot/grub/x86_64-efi/part_msdos.mod
./boot/grub/x86_64-efi/part_plan.mod
./boot/grub/x86_64-efi/part_sun.mod
./boot/grub/x86_64-efi/part_sunpc.mod
./boot/grub/x86_64-efi/parttool.lst
./boot/grub/x86_64-efi/parttool.mod
./boot/grub/x86_64-efi/password.mod
./boot/grub/x86_64-efi/password_pbkdf2.mod
./boot/grub/x86_64-efi/pata.mod
./boot/grub/x86_64-efi/pbkdf2.mod
./boot/grub/x86_64-efi/pbkdf2_test.mod
./boot/grub/x86_64-efi/pcidump.mod
./boot/grub/x86_64-efi/pgp.mod
./boot/grub/x86_64-efi/play.mod
./boot/grub/x86_64-efi/png.mod
./boot/grub/x86_64-efi/priority_queue.mod
./boot/grub/x86_64-efi/probe.mod
./boot/grub/x86_64-efi/procfs.mod
./boot/grub/x86_64-efi/progress.mod
./boot/grub/x86_64-efi/raid5rec.mod
./boot/grub/x86_64-efi/raid6rec.mod
./boot/grub/x86_64-efi/random.mod
./boot/grub/x86_64-efi/rdmsr.mod
./boot/grub/x86_64-efi/read.mod
./boot/grub/x86_64-efi/reboot.mod
./boot/grub/x86_64-efi/regexp.mod
./boot/grub/x86_64-efi/reiserfs.mod
./boot/grub/x86_64-efi/relocator.mod
./boot/grub/x86_64-efi/romfs.mod
./boot/grub/x86_64-efi/scsi.mod
./boot/grub/x86_64-efi/search_fs_file.mod
./boot/grub/x86_64-efi/search_fs_uuid.mod
./boot/grub/x86_64-efi/search_label.mod
./boot/grub/x86_64-efi/search.mod
./boot/grub/x86_64-efi/serial.mod
./boot/grub/x86_64-efi/setjmp.mod
./boot/grub/x86_64-efi/setjmp_test.mod
./boot/grub/x86_64-efi/setpci.mod
./boot/grub/x86_64-efi/sfs.mod
./boot/grub/x86_64-efi/shift_test.mod
./boot/grub/x86_64-efi/shim_lock.mod
./boot/grub/x86_64-efi/signature_test.mod
./boot/grub/x86_64-efi/sleep.mod
./boot/grub/x86_64-efi/sleep_test.mod
./boot/grub/x86_64-efi/smbios.mod
./boot/grub/x86_64-efi/spkmodem.mod
./boot/grub/x86_64-efi/squash4.mod
./boot/grub/x86_64-efi/strtoull_test.mod
./boot/grub/x86_64-efi/syslinuxcfg.mod
./boot/grub/x86_64-efi/tar.mod
./boot/grub/x86_64-efi/terminal.lst
./boot/grub/x86_64-efi/terminal.mod
./boot/grub/x86_64-efi/terminfo.mod
./boot/grub/x86_64-efi/test_blockarg.mod
./boot/grub/x86_64-efi/testload.mod
./boot/grub/x86_64-efi/test.mod
./boot/grub/x86_64-efi/testspeed.mod
./boot/grub/x86_64-efi/tftp.mod
./boot/grub/x86_64-efi/tga.mod
./boot/grub/x86_64-efi/time.mod
./boot/grub/x86_64-efi/tpm.mod
./boot/grub/x86_64-efi/trig.mod
./boot/grub/x86_64-efi/tr.mod
./boot/grub/x86_64-efi/true.mod
./boot/grub/x86_64-efi/udf.mod
./boot/grub/x86_64-efi/ufs1_be.mod
./boot/grub/x86_64-efi/ufs1.mod
./boot/grub/x86_64-efi/ufs2.mod
./boot/grub/x86_64-efi/uhci.mod
./boot/grub/x86_64-efi/usb_keyboard.mod
./boot/grub/x86_64-efi/usb.mod
./boot/grub/x86_64-efi/usbms.mod
./boot/grub/x86_64-efi/usbserial_common.mod
./boot/grub/x86_64-efi/usbserial_ftdi.mod
./boot/grub/x86_64-efi/usbserial_pl2303.mod
./boot/grub/x86_64-efi/usbserial_usbdebug.mod
./boot/grub/x86_64-efi/usbtest.mod
./boot/grub/x86_64-efi/verifiers.mod
./boot/grub/x86_64-efi/video_bochs.mod
./boot/grub/x86_64-efi/video_cirrus.mod
./boot/grub/x86_64-efi/video_colors.mod
./boot/grub/x86_64-efi/video_fb.mod
./boot/grub/x86_64-efi/videoinfo.mod
./boot/grub/x86_64-efi/video.lst
./boot/grub/x86_64-efi/video.mod
./boot/grub/x86_64-efi/videotest_checksum.mod
./boot/grub/x86_64-efi/videotest.mod
./boot/grub/x86_64-efi/wrmsr.mod
./boot/grub/x86_64-efi/xfs.mod
./boot/grub/x86_64-efi/xnu.mod
./boot/grub/x86_64-efi/xnu_uuid.mod
./boot/grub/x86_64-efi/xnu_uuid_test.mod
./boot/grub/x86_64-efi/xzio.mod
./boot/grub/x86_64-efi/zfscrypt.mod
./boot/grub/x86_64-efi/zfsinfo.mod
./boot/grub/x86_64-efi/zfs.mod
./boot/grub/x86_64-efi/zstd.mod
./EFI
./EFI/BOOT
./EFI/BOOT/BOOTX64.EFI
./EFI/BOOT/fbx64.efi
./EFI/BOOT/mmx64.efi
./EFI/grub
./EFI/grub/BOOTX64.CSV
./EFI/grub/grub.cfg
./EFI/grub/grubx64.efi
./EFI/grub/mmx64.efi
./EFI/grub/shimx64.efi
```

```sh
rm ./EFI/BOOT/fbx64.efi
cp ./EFI/grub/grubx64.efi ./EFI/BOOT
mkdir -p ./EFI/ubuntu
mv ./EFI/grub/grub.cfg ./EFI/ubuntu/grub.cfg
rm -rf ./EFI/grub

vim ./boot/grub/grub.cfg

# END: inside chroot

umount "${MPBS}/"*
umount "${MPBS}"

unset MPBS
```

### systemd-boot (OLD)

We prefer using grub as grub can also be used for non UEFI systems and we can load the kernel from BTRFS filesystem.

```
mkdir /tmp/1; mount /dev/sdb1 /tmp/1
bootctl install --esp-path=/tmp/1 --no-variables --make-machine-id-directory=no
```

```
Created "/tmp/1/EFI".
Created "/tmp/1/EFI/systemd".
Created "/tmp/1/EFI/BOOT".
Created "/tmp/1/loader".
Created "/tmp/1/loader/entries".
Created "/tmp/1/EFI/Linux".
Copied "/usr/lib/systemd/boot/efi/systemd-bootx64.efi" to "/tmp/1/EFI/systemd/systemd-bootx64.efi".
Copied "/usr/lib/systemd/boot/efi/systemd-bootx64.efi" to "/tmp/1/EFI/BOOT/BOOTX64.EFI".
Random seed file /tmp/1/loader/random-seed successfully written (512 bytes).
```

```
umount /tmp/1
```

#### Update

```
mkdir /tmp/1; mount /dev/sdb3 /tmp/1
mkdir /tmp/3; mount /dev/sdb3 /tmp/3


mkdir -p /tmp/1/boot/ubuntu.focal/


cp /tmp/3/ubuntu.focal/boot/vmlinuz-5.4.0-26-generic /tmp/1/boot/ubuntu.focal/vmlinuz
cp /tmp/3/ubuntu.focal/boot/initrd.img /tmp/1/boot/ubuntu.focal/initrd.img


cat >/tmp/1/loader/entries/ubuntu.focal.conf <<EOF
title   Ubuntu Focal
linux   /boot/ubuntu.focal/vmlinuz
initrd  /boot/ubuntu.focal/initrd.img
options rootwait root=UUID=c40a464a-f21a-48af-b655-b1e75571788f rootflags=subvol=ubuntu.focal rw
EOF


cat >/tmp/1/loader/loader.conf <<EOF
timeout menu-force
EOF

umount /tmp/3
umount /tmp/1
```
