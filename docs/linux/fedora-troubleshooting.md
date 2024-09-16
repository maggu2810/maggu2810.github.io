Fedora Troubleshooting
===

***Information for Fedora related stuff***

**Author:** *Markus Rathgeb*

---

# Links

* [Upgrading Fedora Using DNF System Plugin](https://docs.fedoraproject.org/en-US/quick-docs/upgrading-fedora-offline/)
* [Non-graphical boot with systemd](https://unix.stackexchange.com/a/164028)
* [Unlocking/Mapping LUKS partitions with the device mapper](https://wiki.archlinux.org/title/dm-crypt/Device_encryption#Unlocking/Mapping_LUKS_partitions_with_the_device_mapper)
* [Disable/blacklist Nouveau nvidia driver](https://linuxconfig.org/how-to-disable-blacklist-nouveau-nvidia-driver-on-ubuntu-20-04-focal-fossa-linux)
* [Fedora NVIDIA](https://rpmfusion.org/Howto/NVIDIA)
* [kmod-nvidia vs akmod-nvidia](https://forums.fedoraforum.org/showthread.php?228557-kmod-nvidia-vs-akmod-nvidia)
* [Rebuild Kernel Modules with Akmods](https://brandonrozek.com/blog/rebuildkernelakmod/)
* [Setting up chroot from a live image in Fedora. Regenerate grub2 for Fedora.](https://gist.github.com/Tamal/73e65bfb0e883e438310c5fe81c5de14)
* [Working with the GRUB 2 Boot Loader](https://docs.fedoraproject.org/en-US/fedora/latest/system-administrators-guide/kernel-module-driver-configuration/Working_with_the_GRUB_2_Boot_Loader/#sec-Reinstalling_GRUB_2)
* [Changes/UnifyGrubConfig](https://fedoraproject.org/wiki/Changes/UnifyGrubConfig)

```shell
cat /boot/efi/EFI/fedora/grub.cfg
```

```
search --no-floppy --fs-uuid --set=dev 11111111-2222-3333-4444-555555555555
set prefix=($dev)/grub2
export $prefix
configfile $prefix/grub.cfg
```

```shell
blkid 
```

```
...
/dev/sda2: UUID="11111111-2222-3333-4444-555555555555" BLOCK_SIZE="4096" TYPE="ext4" PARTLABEL="Linux Boot" PARTUUID="aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee"
...
```

Caution:
If blsdir (perhaps bootloader specification directory) is set, the import of that entries are used from this directory.

```
grub2-editenv -v /boot/grub2/grubenv list
```

```
blsdir=/fedora/loader/entries
```


