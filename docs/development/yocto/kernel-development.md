Yocto - Kernel Development
===

***Yocto based kernel development***

**Author:** *Markus Rathgeb*

---

# Introducton

The linux kernel could be provided by different recipes dependent on your setup, e.g.
* linux-yocto
* linux-fslc
* linux-tegra

To refer to the setup specific kernel we can use "virtual/kernel".

# Build

Build kernel:

```bash
bitbake virtual/kernel
```

# Continue

Trigger a force deploy to continue the building where left:

```bash
bitbake virtual/kernel -f -c deploy
```

# Development Shell

Start a devshell

```bash
bitbake virtual/kernel -c devshell
```

All of the yocto scripts required to build are in the build output directory, $KBUILD_OUTPUT.

```bash
echo $KBUILD_OUTPUT
```

Now you can execute e.g. the make command without consider which ARCH, CC etc. have to be used as all variables are already set.

```bash
make Image
```

# Notes

* [Quick rebuild of device tree only with Yocto/bitbake?](https://stackoverflow.com/questions/38917745/quick-rebuild-of-device-tree-only-with-yocto-bitbake)
* [Compiling a device tree using yocto](https://splefty.blogspot.com/2015/09/compiling-device-tree-using-yocto.html)