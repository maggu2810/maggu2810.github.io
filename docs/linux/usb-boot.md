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

Signed bootloaders will not be modified anymore, so for grub the configuration location is already fix in the signed efi file. Distributions normally use the distribution name (e.g. Ubuntu is using "ubuntu"). So ensure to use correct "bootloader-id" because in this location the grub.cfg will be looked for.

It seems like the signatures of  shim signed + grub signed + kernel signed on Ubuntu are distribution dependent. At least shim + grub of Ubuntu Jammy cannot load a kernel of Ubuntu Focal. Because if this it make sense to place the grub directory inside the specific efi directory, so you can move the directories around if you want to use another one...

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

grub-install --uefi-secure-boot --target=x86_64-efi --boot-directory=/efi/EFI/ubuntu --efi-directory=/efi --bootloader-id=ubuntu --no-nvram
```

```sh
####rm ./EFI/BOOT/fbx64.efi
##cp ./EFI/ubuntu/grubx64.efi ./EFI/BOOT

vim ./EFI/ubuntu/grub/grub.cfg

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
