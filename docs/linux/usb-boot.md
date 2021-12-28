# Short Summary

* https://wiki.debian.org/UEFI
* https://www.thomas-krenn.com/en/wiki/Partition_Alignment_detailed_explanation
* https://wiki.debianforum.de/Ein_Notfallsystem_auf_einem_USB-Stick_installieren

| Partition     | Zweck                    | Größe                                                        | Partitionstyp | Name            |
| ------------- | ------------------------ | ------------------------------------------------------------ | ------------- | --------------- |
| »*/dev/sdz1*« | EFI System Partition     | +2G                                                          | ef00          | EFI.RESCUE      |
| »*/dev/sdz2*« | BIOS Boot Partition      | +100M                                                        | ef02          | BIOSBOOT.RESCUE |
| »*/dev/sdz3*« | Partition für das System | Vorgabe (gesamter noch verfügbarer Speicherplatz, mindestens ~4 GB) | 8300          | BTRFS.RESCUE    |

```
root@debian:~# gdisk /dev/sdz
```

```sh
GPT fdisk (gdisk) version 1.0.1

Partition table scan:
  MBR: protective
  BSD: not present
  APM: not present
  GPT: present


Found valid GPT with protective MBR; using GPT.


Command (? for help): o
This option deletes all partitions and creates a new protective MBR.
Proceed? (Y/N): y


Command (? for help): n
Partition number (1-128, default 1):
First sector (34-60555230, default = 2048) or {+-}size{KMGTP}:
Last sector (2048-60555230, default = 60555230) or {+-}size{KMGTP}: +2G
Current type is 'Linux filesystem'
Hex code or GUID (L to show codes, Enter = 8300): ef00
Changed type of partition to 'EFI System'

Command (? for help): c
Using 1
Enter name: EFI.RESCUE
 
Command (? for help): n
Partition number (2-128, default 2):
First sector (34-60555230, default = 4196352) or {+-}size{KMGTP}:
Last sector (4196352-60555230, default = 60555230) or {+-}size{KMGTP}: +100M
Current type is 'Linux filesystem'
Hex code or GUID (L to show codes, Enter = 8300): ef02
Changed type of partition to 'BIOS boot partition'


Command (? for help): n
Partition number (3-128, default 3):
First sector (34-60555230, default = 4401152) or {+-}size{KMGTP}:
Last sector (4401152-60555230, default = 60555230) or {+-}size{KMGTP}:
Current type is 'Linux filesystem'
Hex code or GUID (L to show codes, Enter = 8300):
Changed type of partition to 'Linux filesystem'
Command (? for help): c
Partition number (1-3): 2
Enter name: BIOSBOOT.RESCUE

Command (? for help): c
Partition number (1-3): 3
Enter name: BTRFS.RESCUE
Command (? for help): p
Disk /dev/sdz: 60555264 sectors, 28.9 GiB
Logical sector size: 512 bytes
Disk identifier (GUID): B6B427CF-881B-4883-8B4A-7C3E3EEC7C76
Partition table holds up to 128 entries
First usable sector is 34, last usable sector is 60555230
Partitions will be aligned on 2048-sector boundaries
Total free space is 2014 sectors (1007.0 KiB)


Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048         4196351   2.0 GiB     EF00  EFI.RESCUE
   2         4196352         4401151   100.0 MiB   EF02  BIOSBOOT.RESCUE
   3         4401152        60555230   26.8 GiB    8300  BTRFS.RESCUE


Command (? for help): wq


Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING
PARTITIONS!!

Do you want to proceed? (Y/N): y
OK; writing new GUID partition table (GPT) to /dev/sdz.
The operation has completed successfully.
```

```
root@debian:~# mkfs.vfat -n EFI.RESCUE /dev/sdz1
root@debian:~# mkfs.btrfs -L BTRFS.RESCUE /dev/sdz1
```

```
root@debian:~# mkdir /mnt/tmp
```

```
root@debian:~# mount /dev/sdz3 /mnt/tmp/
root@debian:~# btrfs subvolume create /mnt/tmp/rescue
root@debian:~# umount /mnt/tmp
```

```
root@debian:~# mount -o subvol=rescue,compress,noatime /dev/sdz3 /mnt/tmp
```

```
root@debian:~# debootstrap --arch=amd64 ... /mnt/tmp
```