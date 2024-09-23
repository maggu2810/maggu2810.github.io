Device Tree
===

***device tree related information***

**Author**: *Markus Rathgeb*

---

# Runtime Load Support

## Variant 1

[dtbocfg - Device Tree Blob Overlay Configuration File System](https://github.com/ikwzm/dtbocfg)

## Variant 2

The "OF: DT-Overlay configfs interface" kernel patches are used by
separate downstream kernels.

See LKMS: [of: Status of DT-Overlay configfs patch](https://lore.kernel.org/lkml/CAMuHMdVuJFRrHAR8Q+HkXbaf29TaUFgvxYY4Ua9xQ7mGZoBsnQ@mail.gmail.com/T/#t)

Add for example the following remotes to your kernel repo
* https://github.com/altera-opensource/linux-socfpga.git
* https://git.kernel.org/pub/scm/linux/kernel/git/geert/renesas-drivers.git
* https://github.com/Xilinx/linux-xlnx.git

This way you can find branches and commits for the relevant patches:

```
git log --oneline --all --source --grep \
       'OF: DT-Overlay configfs interface' | \
  grep 'OF: DT-Overlay configfs interface' | \
  awk '{print $2" "$1}' | \
  sort -V
```
