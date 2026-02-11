Fedora KDE
===

***Information related to Fedora KDE***

**Author:** *Markus Rathgeb*

---

# From Fedora (Gnome) to Fedora KDE

```shell
dnf group list -v --available  
dnf group list -v --installed  
```

```shell 
# as root

dnf upgrade --refresh  

dnf install --allowerasing vim-default-editor  

dnf swap fedora-release-identity-workstation fedora-release-identity-kde  
dnf install @kde-desktop-environment  
dnf install @kde-desktop  
  
systemctl disable gdm.service  
systemctl enable sddm  
  
dnf swap @gnome-desktop @kde-desktop  
dnf swap @workstation-product-environment @kde-desktop-environment  
  
dnf remove @gnome-desktop  
dnf remove @workstation-product-environment  
  
  
dnf install @container-management  
dnf install @firefox; dnf remove fedora-bookmarks  
dnf install @libreoffice; dnf remove libreoffice-gtk\* libreoffice-x11  
  
dnf install \  
@c-development \  
@development-tools \  
@fonts \  
@hardware-support \  
@sound-and-video  
  
  
# read carefully  
dnf remove \*gnome\*  
  
# flatpak  
# Error: Error deploying: Not allowed to query parental controls data for user 1000  
# [https://github.com/flatpak/flatpak/issues/5264](https://github.com/flatpak/flatpak/issues/5264)  
dnf install malcontent  
  
dnf remove '*doublecmd*'  
  
dnf install okular  
dnf install kompare kdiff3 krename krusader  
  
rm -rf ~/.config/krusaderrc ~/.local/share/krusader  
  
#sudo dnf remove meld  
#git config --global merge.tool kdiff3  
#git config --global diff.tool kompare  
git config --global merge.tool meld  
git config --global diff.tool meld  
  
  
# dnf install kio-fuse  
  
  
dnf remove gedit  
  
  
dnf install gwenview  
dnf remove eog  
  
dnf install krdc  
  
  
  
# ----  
  
dnf swap 'Fedora Workstation' @server-product-environment  
dnf swap fedora-release-identity-workstation fedora-release-identity-server  
dnf swap fedora-release-workstation fedora-release-server  
dnf swap @workstation-product-environment @server-product-environment  
  
# ----  
  
dnf list --installed | grep rpmfusion  
dnf swap ffmpeg ffmpeg-free --allowerasing  
dnf upgrade  
  
# ----  
  
dnf install adoptium-temurin-java-repository  
  
dnf install $(rpm -qa '*openjdk*' | sed 's:openjdk-.*$:openjdk:g' | uniq | sed 's:java-\(.*\)-openjdk:temurin-\1-jdk:g')  
  
dnf remove $(rpm -qa '*openjdk*' | grep -v 'java-21-openjdk-headless')
```

# Update to Fedora 44

```
systemctl disable --now display-manager
dnf remove '*sddm*'
systemctl enable --now plasma-login

dnf remove 'maliit*'
```