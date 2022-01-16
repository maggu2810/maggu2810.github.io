# Short Summary

```
export MPBS="/ubuntu.focal"

btrfs subvolume create "${MPBS}"
debootstrap --arch=amd64 focal "${MPBS}"
```

```
systemd-nspawn -D "${MPBS}" --bind-ro /etc/resolv.conf
```

## Inside Container

### root password

```
passwd
```

### apt sources

```
echo 'deb http://archive.ubuntu.com/ubuntu focal main universe restricted multiverse' > /etc/apt/sources.list
```

### hostname

```
echo 'usblin.localdomain' > /etc/hostname
```

```
echo '127.0.0.1    usblin usblin.localdomain' >> /etc/hosts
echo '::1          usblin usblin.localdomain' >> /etc/hosts
```

### Localization

```
echo 'en_US.UTF-8 UTF-8' >> /etc/locale.gen
locale-gen
```

As `localectl set-locales "LANG=en_US.UTF-8"` does not seem to work (there is a bug report that it is handled differently in that distribution), let's do it the distribution way...

```
echo "LANG=en_US.UTF-8" > /etc/default/locale
```

### update

```
apt update && apt upgrade
```

### keyboard

```
apt install console-data
```

On the configuration menu select "Don't touch keymap"

```
dpkg-reconfigure console-data
```

Select "full list" and after that "pc / qwertz / German / Standard / latin1"

```
dpkg-reconfigure keyboard-configuration
```

* Generic 105-key PC (intl.)
* German
* German
* The default for the keyboard layout
* No compose key

```
setupcon
```

#### diff

If seems the changes necessary are applied by keyboard-configuration reconfiguration.

3 Files are changed:

```diff
diff -Naur ubuntu.focal_03-after-reconfigure-console-data/etc/default/keyboard ubuntu.focal_04-after-reconfigure-keyboard-configuration/etc/default/keyboard
--- ubuntu.focal_03-after-reconfigure-console-data/etc/default/keyboard	2021-12-29 23:03:42.733092285 +0100
+++ ubuntu.focal_04-after-reconfigure-keyboard-configuration/etc/default/keyboard	2021-12-29 23:47:28.455662649 +0100
@@ -3,7 +3,7 @@
 # Consult the keyboard(5) manual page.
 
 XKBMODEL="pc105"
-XKBLAYOUT="us"
+XKBLAYOUT="de"
 XKBVARIANT=""
 XKBOPTIONS=""
 
```

```diff
diff -Naur ubuntu.focal_03-after-reconfigure-console-data/etc/console-setup/cached_setup_keyboard.sh ubuntu.focal_04-after-reconfigure-keyboard-configuration/etc/console-setup/cached_setup_keyboard.sh
--- ubuntu.focal_03-after-reconfigure-console-data/etc/console-setup/cached_setup_keyboard.sh	2021-12-29 23:03:52.329101506 +0100
+++ ubuntu.focal_04-after-reconfigure-keyboard-configuration/etc/console-setup/cached_setup_keyboard.sh	2021-12-29 23:47:28.855634759 +0100
@@ -4,5 +4,10 @@
     rm /run/console-setup/keymap_loaded
     exit 0
 fi
-kbd_mode '-u' 
+kbd_mode '-u' < '/dev/tty1' 
+kbd_mode '-u' < '/dev/tty2' 
+kbd_mode '-u' < '/dev/tty3' 
+kbd_mode '-u' < '/dev/tty4' 
+kbd_mode '-u' < '/dev/tty5' 
+kbd_mode '-u' < '/dev/tty6' 
 loadkeys '/etc/console-setup/cached_UTF-8_del.kmap.gz' > '/dev/null' 
```

The last one is a gzipped one and contains the keymap / keycode handling

```diff
diff -Naur ubuntu.focal_03-after-reconfigure-console-data/etc/console-setup/cached_UTF-8_del.kmap.gz ubuntu.focal_04-after-reconfigure-keyboard-configuration/etc/console-setup/cached_UTF-8_del.kmap.gz
--- ubuntu.focal_03-after-reconfigure-console-data/etc/console-setup/cached_UTF-8_del.kmap.gz	2021-12-29 23:03:52.217101398 +0100
+++ ubuntu.focal_04-after-reconfigure-keyboard-configuration/etc/console-setup/cached_UTF-8_del.kmap.gz	2021-12-29 23:47:28.763641174 +0100
```

### install packages

```
apt install shim-signed grub-efi-amd64-signed
```

```
apt install vim
```

```
apt install btrfs-progs
```

```sh
apt install network-manager
# https://bugs.launchpad.net/ubuntu/+source/network-manager/+bug/1658921
touch /etc/NetworkManager/conf.d/10-globally-managed-devices.conf
```

```sh
apt install linux-image-generic
```

```sh
apt install gnome-session-wayland gnome
# apt install gnome-session gdm3 firefox
```

### user

```sh
groupadd --system my_sudo
echo '%my_sudo ALL=(ALL:ALL) ALL' > /etc/sudoers.d/my_sudo
chmod 0440 /etc/sudoers.d/my_sudo

export UGNAME=unpriv
groupadd --gid 1000 "${UGNAME}"
useradd --uid 1000 --gid "${UGNAME}" -m "${UGNAME}"
passwd "${UGNAME}"

for GRP in \
wheel \
my_sudo \
adm dialout cdrom sudo dip plugdev lpadmin lxd sambashare wireshark docker \
; do gpasswd -a "${UGNAME}" "${GRP}"; done

unset UGNAME


## Old Notes

systemd-boot `.../loader/entries/ubuntu.focal.conf`

```
title   Ubuntu Focal
linux   /boot/ubuntu.focal/vmlinuz
initrd  /boot/ubuntu.focal/initrd.img
options rootwait root=UUID=f6cb8681-5db4-459a-93be-3ed99dec17bc rootflags=subvol=ubuntu.focal rw
```

systemd-boot `.../loader/loader.conf`

```
timeout 3
#console-mode keep
default b127226d796f414c97acaa74128fffe0-*
```

fstab

```
UUID="f6cb8681-5db4-459a-93be-3ed99dec17bc" /     btrfs subvol=/ubuntu.focal 0 0
UUID="D350-81FD"                            /efi  vfat  defaults             0 0
/efi/boot/ubuntu.focal                      /boot none  bind                 0 0
```
