Fedora Packaging
===

***Information for Fedora Package Creation***

**Author:** *Markus Rathgeb*

---

# Notes

Assume

* the current directory contains exact one spec file
* our host architecture is amd64
* we want to build for aarch64

Downloading source

```shell
spectool -g *.spec
```

Cross package creation

```shell
fedpkg --release f38 mockbuild --mock-config fedora-38-aarch64
```

```shell
#!/bin/bash

for DIR in qrtr pd-mapper qbootctl #rmtfs tqftpserv
do
  pushd "${DIR}"
  spectool -g *.spec
  fedpkg --release f38 mockbuild --mock-config ./fedora-38-aarch64-specific.cfg
  popd
done
```

```shell
cat qrtr/fedora-38-aarch64-specific.cfg 
```

```text
include('fedora-38-aarch64.cfg')

config_opts['createrepo_on_rpms'] = True
```

```shell
cat pd-mapper/fedora-38-aarch64-specific.cfg 
```

```text
include('fedora-38-aarch64.cfg')

config_opts['createrepo_on_rpms'] = True

#config_opts['nspawn_args'] = config_opts['nspawn_args'] + ['--bind-ro=/home/maggu2810/workspace/projects/linux-mobile/op6/fedora-container/rpms/qrtr/results_qrtr/1.0/1.fc38:/repo/qrtr']
config_opts['nspawn_args'] = config_opts['nspawn_args'] + ['--bind-ro=../qrtr/results_qrtr/1.0/1.fc38:/repo/qrtr']

config_opts['dnf.conf'] = config_opts['dnf.conf'] + """
[qrtr]
name=qrtr
gpgcheck=0
enabled=1
baseurl=file:///repo/qrtr
skip_if_unavailable=False
"""
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
