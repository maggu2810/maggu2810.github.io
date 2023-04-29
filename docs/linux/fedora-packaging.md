Fedora Packaging
===

***Information for Fedora Package Creation***

**Author:** *Markus Rathgeb*

---

# Notes

```shell
fedpkg --release f38 mockbuild --mock-config fedora-38-aarch64
```

# Links

* [Cross Compile Kernel](https://discussion.fedoraproject.org/t/how-does-one-cross-compile-the-fedora-kernel/72574/5)
* Fedora Project Docs
  * [Installing Packager Tools](https://docs.fedoraproject.org/en-US/package-maintainers/Installing_Packager_Tools/)
  * [Packaging Tutorial: GNU Hello](https://docs.fedoraproject.org/en-US/package-maintainers/Packaging_Tutorial_GNU_Hello/)
  * [Using the Koji build system](https://docs.fedoraproject.org/en-US/package-maintainers/Using_the_Koji_Build_System/)
* [Bug 1471593 - mockbuild couldn't support cross compilation ](https://bugzilla.redhat.com/show_bug.cgi?id=1471593)
* [RPM Packaging](https://developer.fedoraproject.org/deployment/rpm/about.html)
* [man: fedpkg(1)](https://linux.die.net/man/1/fedpkg)
