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

For further instructions see e.g. [bootstrap](bootsrap.md)

```
umount /tmp/3
```

## Bootloader

### Grub


Damit das Rettungssystem auf möglichst allen PCs booten kann, werden nacheinander

* `grub-pc` für das Booten auf PCs mit BIOS oder mit UEFI im Legacy-Modus
* `grub-efi-amd64` für PCs mit einem 64-bittigen (U)EFI
* `grub-efi-ia32` für PCs mit einem 32-bittigen (U)EFI (vor allem frühe Computer von Apple mit Intel CPU und einige Systeme mit Intel Atom)

installiert.

We are using an Ubuntu system for this example to install all that stuff.

First unload the EFI related support to ensure no variables are modified:

```
umount /sys/firmware/efi/efivars
rmmod efivarfs
rmmod efi_pstore
rmmod efivars
```

Mount and chroot

```
mkdir -p /tmp/3
mount /dev/sdb3 /tmp/3/ -o subvol=ubuntu.focal
mount -o bind /sys /tmp/3/sys
mount -o bind /dev /tmp/3/dev
mount -o bind /proc /tmp/3/proc
chroot /tmp/3
```

Noch immer in der chroot-Umgebung des Rettungssystems wird nun also ein Verzeichnis zum Mounten der EFI System Partition angelegt, selbige dort gemountet und es werden noch ein paar weitere notwendige Verzeichinsse angelegt

```
mkdir -p /boot/efi
mount /dev/sdb1 /boot/efi
mkdir -p /boot/efi/EFI/boot
mkdir -p /boot/efi/EFI/grub
```

Hinweis: Nicht vergessen, dass sich die Pfadangaben auf das chroot-System, also das Rettungssystem beziehen. Außerhalb der chroot Umgebung wäre als obeispielsweise »/boot/efi/EFI« in »/mnt/tmp/boot/efi/EFI« zu finden

First remove all potential installed grub loaders:

```
dpkg --get-selections | grep grub | awk '{print $1}' | xargs apt purge -y
```

Als letztes werden für jeden der drei Grub-Varianten einzeln und nacheinander das grub-Paket installiert, dann grub selbst installiert und dann das jeweilige Paket wieder deinstalliert. Anders geht es nicht, weil die verschiedenen grub-Pakete in Konflikt miteinander stehen, es hat aber außerdem den Vorteil, dass man sich später, zB bei einem Update des Rettungssystems, den Bootloader nicht irrtümlich überschreiben lassen kann.

Den Anfang macht der heikelste Bootloader Debianpackage.png grub-pc. Man installiert ihn mit

```
DEBIAN_PRIORITY=low apt install grub-pc
```

muss aber aufpassen, dass im darauf folgenden Konfigurationsdialog alle Geräte abgewählt sind und man muss daraufhin noch einmal bestätigen, dass grub tatsächlich auf kein Gerät installiert werden soll. Die restlichen Optionen nach denen gefragt wird spielen weiter keine Rolle und die Vorschläge können einfach bestätigt werden. Erst nach der Installation des Pakets erfolgt die eigentliche Installation von grub(-pc):

```
grub-install --boot-directory=/boot/efi/EFI /dev/sdb
```

Jetzt wird das Paket Debianpackage.png grub-pc also wieder deinstalliert

```
apt purge grub-pc grub-pc-bin
```

und das nächste grub-Paket - Debianpackage.png grub-efi-ia32 für Systeme mit 32-bittigem (U)EFI - wird installiert

```
apt install grub-efi-ia32
```

Wieder folgt die eigentliche grub-Installation — wobei die abgefragten Konfigurationsoptionen wieder keine Rolle spielen und die Vorgaben übernommen werden können

```
grub-install --boot-directory=/boot/efi/EFI --efi-directory=/boot/efi --bootloader-id=grub --no-nvram --target=i386-efi
```

und die Deinstallation

```
apt purge grub-efi-ia32 grub-efi-ia32-bin
```

Schließlich folgt das dritte und letzte grub-Paket Debianpackage.png grub-efi-amd64 für 64-bittige (U)EFIs

```
apt install grub-efi-amd64
```

und wieder spielen die Konfigurationsoptionen keine Rolle. Wieder wird dieser grub installiert

```
ggrub-install --boot-directory=/boot/efi/EFI --efi-directory=/boot/efi --bootloader-id=grub --no-nvram
```

und hinterher das grub-Paket deinstalliert:

```
apt purge grub-efi-amd64 grub-efi-amd64-bin
```

Now edit the configuration file `/boot/efi/EFI/grub/grub.cfg` (it will be used by all variants):

```
search.fs_uuid 6641-AD18 root hd1,gpt1 
set prefix=($root)'/grub'
configfile $prefix/grub.cfg
```

and file `/boot/efi/grub/grub.cfg`:

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

menuentry 'Ubuntu Focal (2)' {
        search --no-floppy --fs-uuid --set=root c40a464a-f21a-48af-b655-b1e75571788f
        linux  /ubuntu.focal/boot/vmlinuz rootwait root=UUID=c40a464a-f21a-48af-b655-b1e75571788f rootflags=subvol=ubuntu.focal rw
        initrd  /ubuntu.focal/boot/initrd.img
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
