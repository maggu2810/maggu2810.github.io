OnePlus 6: Arch Linux ARM
===

***Create Arch Linux ARM images for OnePlus 6***

**Author**: *Markus Rathgeb*

---

# Introduction

The initial version has been taken from [here](https://gitlab.gnome.org/sonny/op6/-/blob/main/instructions.txt)

# Walk Through

## Container Preparation

```shell
wget http://os.archlinuxarm.org/os/ArchLinuxARM-aarch64-latest.tar.gz
dd if=/dev/zero of=archlinux.img bs=1 count=0 seek=10G
mkfs.ext4 archlinux.img
mkdir -p rootfs
sudo mount archlinux.img rootfs
sudo tar -xpvf ArchLinuxARM-aarch64-latest.tar.gz -C rootfs
sudo umount rootfs
```

## Container Pre-Enter

```shell
sudo mount archlinux.img rootfs
sudo mv rootfs/etc/resolv.conf rootfs/etc/resolv.conf.bak
sudo cp /etc/resolv.conf rootfs/etc/resolv.conf
```

## Container Enter

```shell
#sudo -s
#mount --bind /dev rootfs/dev
#mount -t devpts devpts rootfs/dev/pts -o gid=5,mode=620
#mount -t proc proc rootfs/proc
#mount -t sysfs sysfs rootfs/sys
#mount -t tmpfs tmpfs rootfs/run
#chroot rootfs /usr/bin/bash

sudo systemd-nspawn -D rootfs/
```

## Work Inside Container

Now we have a root shell in the change rooted Arch Linux ARM (aarch64) root filesystem.

```shell
echo "oneplus6" > /etc/hostname

echo 'root:123456' | chpasswd 

ln -sf /usr/share/zoneinfo/Europe/Berlin /etc/localtime

sed 's:^#en_US.UTF-8 UTF-8:en_US.UTF-8 UTF-8:g' -i /etc/locale.gen
locale-gen
echo 'LANG="en_US.UTF-8"' > /etc/locale.conf

. /etc/locale.conf

pacman-key --init
pacman-key --populate archlinuxarm

# uninstall kernel, we'll use our own anyway
pacman -R --noconfirm linux-aarch64

pacman -Syu --noconfirm

echo 'alarm:123456' | chpasswd 
usermod -aG wheel alarm

pacman -S --noconfirm --needed \
  sudo
sed 's:^# %wheel ALL=(ALL\:ALL) ALL:%wheel ALL=(ALL\:ALL) ALL:g' -i /etc/sudoers

pacman -S --noconfirm --needed \
  gnome-shell gdm \
  networkmanager modemmanager \
  pipewire pipewire-pulse wireplumber \
  bluez bluez-utils gnome-control-center gnome-console \
  linux-firmware-qcom \
  gnome-software flatpak gnome-backgrounds \
  bash-completion \
  iio-sensor-proxy \
  xdg-user-dirs xdg-desktop-portal-gnome

systemctl enable gdm NetworkManager ModemManager bluetooth sshd

# stuff needed for qcom op6, to quote caleb:
# rmtfs provides a service the modem uses to interact with the filesystem,
# tqftpserv allows browsing the filesystem (?) and
# pd-mapper is some other funky shit. embedded platform go brrr

pacman -S --noconfirm --needed \
  qrtr rmtfs
systemctl enable qrtr-ns rmtfs

# enable installing software to /usr/local
echo -n '' > /etc/profile.d/zzz-environment.sh
chmod 0755 /etc/profile.d/zzz-environment.sh
echo "export LD_LIBRARY_PATH=/usr/local/lib" >> /etc/profile.d/zzz-environment.sh
echo "export PKG_CONFIG_LIBDIR=/usr/local/lib/pkgconfig:/usr/local/share/pkgconfig:/usr/lib/pkgconfig:/usr/share/pkgconfig" >> /etc/profile.d/zzz-environment.sh

# build software for op6
pacman -S --noconfirm --needed \
  gdm \
  git ccache meson base-devel \
  xorg-server-xvfb sysprof asciidoc \
  wayland-protocols gobject-introspection \
  evolution-data-server sassc itstool \
  cmake \
  rsync cpio

git config --system user.email "you@example.com"
git config --system user.name "Your Name"
```

```shell
su -l -c '. /etc/profile
git clone https://gitlab.gnome.org/maggu2810/op6.git /home/alarm/git/op6
' alarm
```

```shell
su -l -c '. /etc/profile
git clone https://github.com/andersson/pd-mapper.git /home/alarm/git/pd-mapper
cd /home/alarm/git/pd-mapper
make
' alarm

(cd /home/alarm/git/pd-mapper; make install)
systemctl enable pd-mapper
```

```shell
su -l -c '. /etc/profile
git clone https://github.com/andersson/tqftpserv.git /home/alarm/git/tqftpserv
cd /home/alarm/git/tqftpserv
make
' alarm

(cd /home/alarm/git/tqftpserv; make install)
systemctl enable tqftpserv
```

```shell
# grrrr, modemmanager on arch for some reason disables support for qcom devices

su -l -c '. /etc/profile
git clone https://gitlab.freedesktop.org/mobile-broadband/ModemManager.git /home/alarm/git/ModemManager
cd /home/alarm/git/ModemManager
# even more grrr, arch ships too old deps for mm from main
git checkout 1.20.6
meson build
meson compile -C build
' alarm

(cd /home/alarm/git/ModemManager/build; ninja install)
```

```shell
# qbootctl is some tool that talks to qcom api to tell it that the boot was a success
# without it device will sometimes go into coredump mode

su -l -c '. /etc/profile
git clone https://gitlab.com/sdm845-mainline/qbootctl /home/alarm/git/qbootctl
cd /home/alarm/git/qbootctl
meson build
meson compile -C build
' alarm

(cd /home/alarm/git/qbootctl/build; ninja install)
```

```shell
install -o 0 -g 0 -m 0644 \
  /home/alarm/git/op6/src/mark-boot-successful.service \
  /usr/local/lib/systemd/system/mark-boot-successful.service
systemctl enable mark-boot-successful
```

```shell
# yay, one more thing to get modem working:
# modemmanager doesn't support multi-sim,
# so op6 needs a weird script to run at boot for provisioning the sim card
install -o 0 -g 0 -m 0755 \
  /home/alarm/git/op6/src/op6-provision-simcard.sh \
  /usr/local/bin/op6-provision-simcard.sh
install -o 0 -g 0 -m 0644 \
  /home/alarm/git/op6/src/op6-provision-simcard.service \
  /usr/local/lib/systemd/system/op6-provision-simcard.service
systemctl enable op6-provision-simcard
```

```shell
# this almost looks good compared to the stuff we did before
# change two gsettings to make modal dialogs usable
su -l -c '. /etc/profile
gsettings set org.gnome.desktop.wm.preferences button-layout 'appmenu:close'
gsettings set org.gnome.mutter attach-modal-dialogs false
' alarm
```

```shell
# install firmware from postmarket package

echo '
export FILE="firmware-oneplus-sdm845-7-r1.apk"
mkdir -p /home/alarm/workspace/firmware-oneplus-sdm845
cd /home/alarm/workspace/firmware-oneplus-sdm845
curl -O "https://mirror.postmarketos.org/postmarketos/master/aarch64/${FILE}"
mkdir root
tar -zxvf "${FILE}" -C root
mv root/lib/firmware/postmarketos/ lib-firmware-postmarketos
mv root/lib/firmware/ lib-firmware
' | su -l  -- alarm -i -l

chown -R 0:0 /home/alarm/workspace/firmware-oneplus-sdm845/lib-firmware/
chown -R 0:0 /home/alarm/workspace/firmware-oneplus-sdm845/lib-firmware-postmarketos/
cp -ax /home/alarm/workspace/firmware-oneplus-sdm845/lib-firmware/* /lib/firmware/
cp -ax /home/alarm/workspace/firmware-oneplus-sdm845/lib-firmware-postmarketos/* /lib/firmware/
```

```shell
echo '
git clone https://gitlab.com/sdm845-mainline/alsa-ucm-conf /home/alarm/git/alsa-ucm-conf
cd /home/alarm/git/alsa-ucm-conf
# every commit after this commit does not work on arch :(
git checkout 9a69297d
' | su -l  -- alarm -i -l

for i in ucm ucm2; do
  chown -R 0:0 /home/alarm/git/alsa-ucm-conf/${i}/
  rsync -a /home/alarm/git/alsa-ucm-conf/${i}/ /usr/share/alsa/${i}/
done
```

build shell, mutter, gdm from mobile-shell branch

```shell
# build shell, mutter, gdm from mobile-shell branch

# because of
# ERROR: Dependency lookup for glib-2.0 with method 'pkgconfig' failed: Invalid version, need 'glib-2.0' ['>= 2.75.1'] found '2.74.6'.
# revert: 49b0a892 and f34c6da9

echo '
git clone https://gitlab.gnome.org/verdre/mutter.git /home/alarm/git/mutter
cd /home/alarm/git/mutter
git checkout mobile-shell-devel
git revert \
  49b0a892 \
  f34c6da9
meson build
meson compile -C build
' | su -l  -- alarm -i -l

(cd /home/alarm/git/mutter/build; ninja install)
```

```shell
echo '
git clone https://gitlab.gnome.org/verdre/gnome-shell.git /home/alarm/git/gnome-shell
cd /home/alarm/git/gnome-shell
git checkout mobile-shell-devel
meson build
meson compile -C build
' | su -l  -- alarm -i -l

(cd /home/alarm/git/gnome-shell/build; ninja install)
```

```shell
echo '
git clone https://gitlab.gnome.org/GNOME/gdm.git /home/alarm/git/gdm
cd /home/alarm/git/gdm
meson build
meson compile -C build
' | su -l  -- alarm -i -l

(cd /home/alarm/git/gdm/build; ninja install)
```

```shell
# install kernel from postmarket package
echo '
export FILE="linux-postmarketos-qcom-sdm845-6.2.0-r1.apk"
mkdir -p /home/alarm/workspace/kernel
cd  /home/alarm/workspace/kernel
curl -O https://mirror.postmarketos.org/postmarketos/master/aarch64/"${FILE}"
mkdir root
tar -zxvf "${FILE}" -C root
' | su -l  -- alarm -i -l

chown -R 0:0 /home/alarm/workspace/kernel/root/
rm -rf \
  /boot/* \
  /lib/modules/* \
  /usr/share/kernel/*
for i in boot lib usr; do
  cp -ax /home/alarm/workspace/kernel/root/${i}/* /${i}/
done
mv /boot/vmlinuz /boot/vmlinuz-6.2.0-sdm845

# and here comes the real fun

mkinitcpio --generate /boot/initrd.img-6.2.0-sdm845 --kernel 6.2.0-sdm845

# change the initramfs/init a bit

rm -rf /initramfs.tmp
mkdir /initramfs.tmp
(cd /initramfs.tmp/; zcat /boot/initrd.img-6.2.0-sdm845 | cpio -idmv)

# comment out rdlogger_start and add this instead
# exec > /dev/kmsg // for debugging
sed 's:^rdlogger_start:#rdlogger_start\nexec > /dev/kmsg:g' -i /initramfs.tmp/init

# comment out rdlogger_stop
sed 's:^rdlogger_stop:#rdlogger_stop:g' -i /initramfs.tmp/init

# add this to initramfs/init before: rootdev=$(resolve_device "$root") && root=$rootdev
#rwopt="rw"
#root="/dev/sda17"
sed 's:^\(.*\)\(rootdev="$(resolve_device "$root")" && root="$rootdev".*\):\1rwopt="rw"\n\1root="/dev/sda17"\n\1\2:g' -i /initramfs.tmp/init

# done, now recreate the initramfs
(cd /initramfs.tmp/; find . | cpio -o -c -R root:root | gzip -9 > /boot/initrd.img-6.2.0-sdm845)
chmod 0644 /boot/initrd.img-6.2.0-sdm845
```

```shell
# and do all the shit with the pmos tooling

pacman -S --noconfirm --needed \
  pmbootstrap android-tools

echo '
mkdir -p /home/alarm/workspace/pmtools
cd /home/alarm/workspace/pmtools
curl -O "https://gitlab.com/sdm845-mainline/pmtools/-/raw/main/utils/mkbootimg.sh"
chmod +x mkbootimg.sh

echo '/home/alarm/.local/var/pmbootstrap
edge
oneplus
enchilada
y
alarm
gnome-mobile
n
none
y
en_US.UTF-8
oneplus6
n
y' | pmbootstrap init

mkdir -p /home/alarm/pmos
./mkbootimg.sh -d /boot/dtbs/qcom/sdm845-oneplus-enchilada.dtb -r /boot/initrd.img-6.2.0-sdm845 -k /boot/vmlinuz-6.2.0-sdm845 -c root=/dev/sda17
' | su -l  -- alarm -i -l
```

## Container Exit

Leave the container

```shell
exit
```

## Container Copy Content

```shell
cp rootfs/home/alarm/pmos/mainline-boot.img .
```

## Container Post-Exit

```shell
sudo rm rootfs/etc/resolv.conf
sudo mv rootfs/etc/resolv.conf.bak rootfs/etc/resolv.conf

# TODO cleanup:
# - pacman cache
# - ccache
# - bash-history
# - user alarm: git directory
# - user alarm: workspace directory
# - packages for building only

sudo umount -R rootfs/
```

## Flashing

```shell
fastboot flash boot mainline-boot.img
fastboot flash userdata archlinux.img
fastboot reboot
```
