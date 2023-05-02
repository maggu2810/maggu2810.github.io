OnePlus 6: Fedora
===

***Create Fedora images for OnePlus 6***

**Author**: *Markus Rathgeb*

---

# Introduction

... for the moment see [Arch Liunx ARM instructions](op6-instructions-alarm.md) ...

# Software

## Create Packages

Not found on https://packages.fedoraproject.org/

* pd-mapper
    * https://github.com/andersson/pd-mapper.git
        * copr: jforbes/fedora-x13s
        * copr: ahalaney/pd-mapper
    * 'pd-mapper' is the reference implementation for Qualcomm's Protection Domain mapper service.
    * It is required for userspace applications to access the various remote processors (Wi-Fi, modem, sensors...) on recent Qualcomm SoCs using the QRTR protocol.
    * ?? pd-mapper is some other funky shit. embedded platform go brrr ??
* rmtfs
    * https://github.com/andersson/rmtfs
        * copr: missing
    * Qualcomm Remote Filesystem Service Implementation
    * ?? rmtfs provides a service the modem uses to interact with the filesystem ??
* qrtr
    * https://github.com/andersson/qrtr
        * copr: jforbes/fedora-x13s
    * Qualcomm IPC Router support
    * Userspace reference for net/qrtr in the Linux kernel
* tqftpserv
    * https://github.com/andersson/tqftpserv.git
        * copr: missing
    * Trivial File Transfer Protocol server over AF_QIPCRTR
    * ?? tqftpserv allows browsing the filesystem (?) and ??
* qbootctl
    * https://gitlab.com/sdm845-mainline/qbootctl
        * copr: missing
    * A port of the Qualcomm Android bootctrl HAL for musl/glibc userspace.
    * Qualcomm bootctl HAL for Linux
    * qbootctl is some tool that talks to qcom api to tell it that the boot was a success
    * without it device will sometimes go into coredump mode
        * https://gitlab.gnome.org/maggu2810/op6.git
        * enable system service op6/src/mark-boot-successful.service

## Check Build Configuration etc.

* ModemManager
    * grrrr, modemmanager on arch for some reason disables support for qcom devices
    * even more grrr, arch ships too old deps for mm from main
    * yay, one more thing to get modem working:
    * modemmanager doesn't support multi-sim,
    * so op6 needs a weird script to run at boot for provisioning the sim card
    * https://gitlab.freedesktop.org/mobile-broadband/ModemManager.git
        * https://gitlab.gnome.org/maggu2810/op6.git
        * (op6/src/op6-provision-simcard.sh +) enable service op6/src/op6-provision-simcard.service 
