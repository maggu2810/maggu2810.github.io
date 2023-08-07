postmarketOS
===

***postmarketOS related information***

**Author:** *Markus Rathgeb*

---

# Testing

Quoted from: https://matrix.to/#/!xgUMIsYnSIXklwbhrG:postmarketos.org/$lppT9kvSnpbqGiL_B4ozJX7R3mC5xRtB9MEWclZCJH0?via=matrix.org&via=kde.org&via=postmarketos.org

## Kernel

if you want to test only one of our kernel branches/changes, you can

* install `pmbootstrap` using `git clone` method
* use `envkernel` to compile locally
* `pmbootstrap sideload` them to your device to test it.

See:
* https://wiki.postmarketos.org/wiki/Pmbootstrap#From_git
* https://wiki.postmarketos.org/wiki/Compiling_kernels_with_envkernel.sh

after sourcing `envkernel`,
you would have to run `make defconfig sdm845.config` and
`make -j$(nrpoc)` to compile the kernel.

## pmaports MR

to test pmaports MRs like the sensor's one,
wait for the MR's CI pipeline to complete building.
then use `mrtest` in your phone to try out pmaports MR.

https://wiki.postmarketos.org/wiki/Mrtest
