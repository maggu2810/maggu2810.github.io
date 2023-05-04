OnePlus 6 / 6T
===

***OnePlus 6 / 6T (software) modifications***

**Author**: *Markus Rathgeb*

---

# Introduction

This currently serves as a collection of some thoughts for me.

The hardware OP6 and OP6T should be very similar, so I assume the most stuff should be identical.

The hardware I ordered is a OP6 and not a OP6T. So I cannot judge if the information on this site fits to both devices.

Just to be on the safe side: I do not take any guarantees. Do whatever you want to do with this information. It is all your responsibility.

# Codename

| Device Name   | Codename  |
|---------------|:---------:|
| OnePlus 6     | enchilada |
| OnePlus 6T    |  fajita   |

# Roadmap

* order a OnePlus 6 / 6T (e.g. ebay)
* check if all is working
  * touch
  * display
  * sound
  * call
  * sms
  * usb type c jack
    * charging
    * data connection
      * file transfer
      * adb
* update to most recent  android version (so, firmware is up to date)
* [Unlocking the bootloader](#unlocking-the-bootloader)
* [Ensuring all firmware partitions are consistent](#ensuring-all-firmware-partitions-are-consistent)
* custom android rom (optional)
  * lineageos
    * OnePlus 6: [LineageOS enchilada](https://wiki.lineageos.org/devices/enchilada/)
    * OnePlus 6T: [LineageOS fajita](https://wiki.lineageos.org/devices/fajita/)
  * root device
  * collect information about display and touch device
    * i2c
    * evtest
    * dmesg
    * ...
    * get hw_version
* test postmarketos
* try Arch Linux ARM
  * [Instructions Sonny](https://gitlab.gnome.org/sonny/op6/-/blob/main/instructions.txt)
* test fedora minimal rootfs image using kernel from postmarketos or ALARM
  * as it seems (see "Fedora Mobility" Matrix chat) Fedora userspace gets started it could make sense to modify the rootfs so `init` is a simple script that:
    * starts wpa_supplication with a configuration for the home WiFi (so it connects automatically to it)
    * starts an ssh daemon
  * enhance fedora rootfs image

# Special boot modes

## Recovery

* With the device powered off, hold `Volume Down` + `Power`.
* `adb reboot recovery`

## Bootloader / Fastboot / Download

* With the device powered off, hold `Volume Up` + `Power`.
* `adb reboot bootloader`

# Unlocking the bootloader

1. Enable OEM unlock in the Developer options under device Settings, if present.
    1. Open setting, go to "About" and tap on the "Build number" box ~10 times until the "You are now a developer" toast message appears.
    2. Go back to the main settings page, go to "System" and then "Developer options" (it might be hiding behind a dropdown menu). Toggle the switch to "Enable OEM unlocking"
2. Connect the device to your PC via USB.
3. [Enter Download Mode](#bootloader--fastboot--download)
4. Once the device is in fastboot mode, verify your PC finds it by typing: `fastboot devices`
<br/>If you don't get any output or an error:
    * on Windows: make sure the device appears in the device manager without a triangle. Try other drivers until the command above works!
    * on Linux or macOS: If you see no permissions fastboot try running fastboot as root. When the output is empty, check your USB cable and port!
5. Now type the following command to unlock the bootloader: `fastboot oem unlock`
<br>*Note: At this point the device may display on-screen prompts which will require interaction to continue the process of unlocking the bootloader. Please take whatever actions the device asks you to to proceed.*
6. If the device doesn't automatically reboot, reboot it. It should now be unlocked.
7. Since the device resets completely, you will need to re-enable USB debugging to continue.

See also:

* [Information (LineageOS)](https://wiki.lineageos.org/devices/enchilada/install#unlocking-the-bootloader)
* [Information (postmarketOS)](https://wiki.postmarketos.org/wiki/OnePlus_6_(oneplus-enchilada)#Unlock_the_bootloader)

# Ensuring all firmware partitions are consistent

In some cases, the inactive slot can be unpopulated or contain much older firmware than the active slot, leading to various issues including a potential hard-brick. We can ensure none of that will happen by copying the contents of the active slot to the inactive slot.

To do this, sideload the `copy-partitions-20220613-signed.zip` package by doing the following:

1. Download the `copy-partitions-20220613-signed.zip` file from [here](https://mirrorbits.lineageos.org/tools/copy-partitions-20220613-signed.zip). It should have a MD5 sum of `79f2f860830f023b7030c29bfbea7737` or a SHA-256 sum of `92f03b54dc029e9ca2d68858c14b649974838d73fdb006f9a07a503f2eddd2cd`.
2. Sideload the `copy-partitions-20220613-signed.zip` package:
    * On the device, select "Apply Update", then "Apply from ADB" to begin sideload.
    * On the host machine, sideload the package using: `adb sideload copy-partitions-20220613-signed.zip`
3. Now reboot to recovery by tapping "Advanced", then "Reboot to recovery".

Installing LineageOS from recovery

See also:

* [Information (LineageOS)](https://wiki.lineageos.org/devices/enchilada/install#ensuring-all-firmware-partitions-are-consistent)
* [Information (XDA Developers)](https://forum.xda-developers.com/t/rom-13-lineageos-20-0-unofficial-oneplus-6-6t-gapps-ota-updates-safetynet-twrp.4494053/)

# Restore Partitions / Update Firmware

postmarketOS installation instructions states that you should erase the dtbo partition using `fastboot erase dtbo`.

> Erasing the dtbo partition will make Android (and ALL Android-based software like Ubuntu Touch or TWRP recovery) unbootable on the current slot (read this page if you're not sure what that means). You can re-flash an Android ROM via fastboot by extracting the payload.bin from the OTA zip and using a tool like android-ota-payload-extractor to get the individual partition images. It is almost never necessary to resort to extreme measures like "MSM Download tool" to reflash the device via EDL

* Download full stock ROM from [oneplus.com](https://oneplus.com/support/softwareupgrade)
* Extract stock ROM using [payload-dumper-go](https://github.com/ssut/payload-dumper-go)
* ...

See also:

* [Update firmware on enchilada](https://wiki.lineageos.org/devices/enchilada/fw_update)

# Fedora

## Setup

To run aarch64 executables we need to ensure the respective qemu-user-static package is installed and binfmt is setup correctly.

```
dnf install qemu-user-static-aarch64
```

The file `/proc/sys/fs/binfmt_misc/qemu-aarch64` should exist.

If there is something wrong, check the status end if desired restart the service.

```shell
systemctl status systemd-binfmt.service 
systemctl restart systemd-binfmt.service
```