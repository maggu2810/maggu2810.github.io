OnePlus 6 / 6T
===

***OnePlus 6 / 6T (software) modifications***

**Author**: *Markus Rathgeb*

---

# Codename

| Device Name   | Codename  |
|---------------|:---------:|
| OnePlus 6     | enchilada |
| OnePlus 6T    |  fajita   |

# Roadmap

* order a OnePlus 6 / 6T (e.g. ebay)
* check if all is working
  * charging
  * touch
  * display
  * sound
  * call
  * sms
* update to most recent  android version (so, firmware is up to date)
* unlock bootloader
  * https://wiki.lineageos.org/devices/enchilada/install#unlocking-the-bootloader
  * https://wiki.postmarketos.org/wiki/OnePlus_6_(oneplus-enchilada)#Unlock_the_bootloader
* Ensuring all firmware partitions are consistent
  * https://wiki.lineageos.org/devices/enchilada/install#ensuring-all-firmware-partitions-are-consistent
  * https://forum.xda-developers.com/t/rom-13-lineageos-20-0-unofficial-oneplus-6-6t-gapps-ota-updates-safetynet-twrp.4494053/
* custom android rom (optional)
  * lineageos
    * OnePlus 6: https://wiki.lineageos.org/devices/enchilada/
    * OnePlus 6T: https://wiki.lineageos.org/devices/fajita/
  * root device
  * collect information about display and touch device
    * i2c
    * evtest
    * dmesg
    * ...
    * get hw_version
* test postmarketos
* try Arch Linux ARM
  * https://gitlab.gnome.org/sonny/op6/-/blob/main/instructions.txt
* test fedora minimal rootfs image using kernel from postmarketos or ALARM
* enhance fedora rootfs image