sudo dnf install zypper

# opensuse root
export OSR="/home/maggu2810/tmp/opensuse/"

mkdir -p "${OSR}"

# https://en.opensuse.org/Package_repositories
# https://en.opensuse.org/Additional_package_repositories
zypper --root "${OSR}" addrepo http://download.opensuse.org/distribution/leap/15.3/repo/oss/ OSS
zypper --root "${OSR}" addrepo http://download.opensuse.org/update/leap/15.3/oss/ Update

zypper --root "${OSR}" refresh

zypper --root "${OSR}" install \
  aaa_base aaa_base-extras \
  zypper \
  shim-susesigned grub2-x86_64-efi

systemd-nspawn -D "${OSR}"

zypper install \
  aaa_base aaa_base-extras \
  zypper \
  shim-susesigned grub2-x86_64-efi

# rpm -qa | grep grub
# strings /usr/share/efi/x86_64/*.efi
zypper install binutils
