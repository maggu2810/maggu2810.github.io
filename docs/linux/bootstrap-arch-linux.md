# Short Summary

* Download bootstrap image
* https://archlinux.org/download/
* ...

```
mkdir -p /mnt/sdb3
mount /dev/sdb3 /mnt/sdb3
cd /mnt/sdb3
btrfs subvolume create archlinux
umount /mnt/sdb3

mkdir -p /mnt/archlinux
mount /dev/sdb3 -o subvol=archlinux /mnt/archlinux

tar xzf /home/maggu2810/Downloads/archlinux-bootstrap-2022.01.01-x86_64.tar.gz --numeric-owner --strip-components=1 -C /mnt/archlinux
cd /mnt/archlinux
```

## Mirror

Uncomment one mirror:

```
vim etc/pacman.d/mirrorlist
```

## chroot

```
./bin/arch-chroot ./
```

## Time zone

```
ln -sf /usr/share/zoneinfo/Europe/Berlin /etc/localtime
```

## Localization

```
echo 'en_US.UTF-8 UTF-8' >> /etc/locale.gen
locale-gen
```

```
echo "LANG=en_US.UTF-8" > /etc/locale.conf
```

```
echo "KEYMAP=de-latin1" > /etc/vconsole.conf
```

## hostname

```
echo 'usblin.localdomain' > /etc/hostname
```

```
echo '127.0.0.1    usblin usblin.localdomain' >> /etc/hosts
echo '::1          usblin usblin.localdomain' >> /etc/hosts
```

## root password

```
passwd
```

## Initializing pacman keyring

```
pacman-key --init
pacman-key --populate archlinux
```

## Workaround while in chroot

Use this only if really necessary (e.g. if you chroot not into a mount point)
```
sed 's:^CheckSpace$:#CheckSpace:g' -i /etc/pacman.conf
```

## Update

```
pacman -Syu
```

## Install stuff

```
pacman -S --needed btrfs-progs vim networkmanager
```

```
pacman -S --needed base linux linux-firmware
```

## Bootup

After the bootup setup services

```
systemctl enable --now systemd-resolved.service
ln -sf /run/systemd/resolve/stub-resolv.conf /etc/resolv.conf

systemctl enable --now systemd-networkd.service

systemctl enable --now NetworkManager.service
```