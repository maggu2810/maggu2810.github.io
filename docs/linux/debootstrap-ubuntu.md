# Short Summary

* setup sources.list

  * **Main** - Canonical-supported free and open-source software. 
  * **Universe** - Community-maintained free and open-source software. 
  * **Restricted** - Proprietary drivers for devices. 
  * **Multiverse** - Software restricted by copyright or legal issues. 

* install vim

* set hostname

* update /etc/hosts for hostname to localhost IP (IPv4 + IPv6)

* locale.gen (en_US.UTF-8)

* locale-gen

* localectl set-locales "LANG=en_US.UTF-8"

* Locales and Keyboard

  * apt install console-data

  * dpkg-reconfigure console-data [keyboard-configuration]

  * ```
    # apt install console-setup
    # dpkg-reconfigure keyboard-configuration 
    ```

  * loadkeys de-latin1

* mkdir /efi

* apt install linux-image-generic

* apt install gnome-session gdm3 gnome-session-wayland gnome firefox

* NetworkManager

  * bugs.lauchpad.net/ubuntu/+source/network-manager/+bug/1658921
  * touch /etc/NetworkManager/conf.d/10-globally-managed-devices.conf

* Bootloader:

  * kernel-command line

* add fstab content