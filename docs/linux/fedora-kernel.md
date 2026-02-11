Fedora Kernel
===

***Information for Fedora Kernel related stuff***

**Author:** *Markus Rathgeb*

---

# Links

* [Installing Kernel from Koji: Download and install the kernel](https://docs.fedoraproject.org/en-US/quick-docs/kernel-installing-from-koji/#_download_and_install_a_kernel_using_the_koji_client)
* [Koji: search for kernel](https://koji.fedoraproject.org/koji/search?terms=kernel-6.11*fc41*&type=build&match=glob)
* [Fedora Wiki: Building a custom kernel: Building the kernel](https://fedoraproject.org/wiki/Building_a_custom_kernel#Building_the_kernel)
* [Gentoo Wiki: Kernel git-bisect](https://wiki.gentoo.org/wiki/Kernel_git-bisect)

# Install Kernel using koji

```
mkdir -p ~/workspace/oss/fedora/koji/kernel
cd ~/workspace/oss/fedora/koji/kernel

# create list of available kernels
koji search rpm "kernel*" > rpm-kernel.list
cat rpm-kernel.list | sort -V > rpm-kernel-sorted.list

# check for specific version
cat kernel-rpm-sorted.list | grep kernel-6.18

# set variables
export ARCH=x86_64
export VERSION=6.18.9-200.fc43

# download kernel packages
koji download-build --arch="${ARCH}" --arch=noarch kernel-"${VERSION}"

# install related kernel packages
sudo dnf install $(for PKG in kernel kernel-core kernel-devel kernel-modules kernel-modules-core kernel-modules-extra kernel-modules-internal; do echo ${PKG}-${VERSION}.${ARCH}.rpm; done)
```
