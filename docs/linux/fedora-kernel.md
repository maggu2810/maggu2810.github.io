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

```
cd ~/workspace/oss/fedora/koji/

export KERNEL_ID="kernel-6.11.0-0.rc5.43.fc41"
mkdir "${KERNEL_ID}"
cd "${KERNEL_ID}"

koji download-build --arch=x86_64 --arch=noarch "${KERNEL_ID}"

sudo dnf install $(ls *"${KERNEL_ID}"* | grep -v -e '.*debug.*\.rpm' -e '.*uki.*\.rpm')
```
