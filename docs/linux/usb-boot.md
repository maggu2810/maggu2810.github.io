# Short Summary

* https://wiki.debian.org/UEFI
* https://www.thomas-krenn.com/en/wiki/Partition_Alignment_detailed_explanation
* https://wiki.debianforum.de/Ein_Notfallsystem_auf_einem_USB-Stick_installieren

| Partition     | Zweck                    | Größe                                                        | Partitionstyp | Name            |
| ------------- | ------------------------ | ------------------------------------------------------------ | ------------- | --------------- |
| »*/dev/sdb1*« | EFI System Partition     | +2G                                                          | ef00          | EFI_USBLIN      |
| »*/dev/sdb2*« | BIOS Boot Partition      | +100M                                                        | ef02          | BIOSBOOT_USBLIN |
| »*/dev/sdb3*« | Partition für das System | Vorgabe (gesamter noch verfügbarer Speicherplatz, mindestens ~4 GB) | 8300          | BTRFS_USBLIN    |

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

Ungplug and plug in the USB stick.

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

### Grub

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
mount /dev/sdb1 /efi
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
