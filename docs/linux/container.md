Container
===

***Container Management and Handling***

**Author:** *Markus Rathgeb*

---

# Introduction

Generally I prefer the usage of podman rootless.
As not all software already supports the rootless approach, I use podman rootless for stuff that works with and docker
system daemon where this is really necessary only.

# Docker

## Installation

First ensure all old versions are removed

```shell
sudo dnf remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine
```

Now let's install docker

```shell
sudo dnf -y install dnf-plugins-core

sudo dnf config-manager \
    --add-repo \
    https://download.docker.com/linux/fedora/docker-ce.repo

sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

sudo systemctl enable --now docker.socket
```

## Notes

### Fedora 38

At the time of writing (2022-02-23) Fedora 38 (not yet released) is not supported.
But using the Fedora 37 branch of the docker repository works.

```diff
--- /etc/yum.repos.d/docker-ce.repo.org	2023-03-01 19:57:59.964464101 +0100
+++ /etc/yum.repos.d/docker-ce.repo	2023-03-01 19:58:11.927458210 +0100
@@ -1,6 +1,6 @@
 [docker-ce-stable]
 name=Docker CE Stable - $basearch
-baseurl=https://download.docker.com/linux/fedora/$releasever/$basearch/stable
+baseurl=https://download.docker.com/linux/fedora/37/$basearch/stable
 enabled=1
 gpgcheck=1
 gpgkey=https://download.docker.com/linux/fedora/gpg
```

### docker-compose

We do not need to install `docker-compose` using the package manager, pip or another approach. As it has been integrated
into upstream docket by the compose plugin. Instead of `docker-compose` use `docker compose` (replace the hyphen with a
space).

If there would be any need to have the "old" docker-compose available:

```shell
python3 -m venv "${HOME}/bin/pkgs/docker-compose-venv"
. "${HOME}/bin/pkgs/docker-compose-venv/bin/activate"
pip install --upgrade pip
pip install docker-compose
```

## Uninstall

```shell
sudo systemctl disable --now docker.socket docker.service

sudo dnf remove `rpm -qa '*docker*'` containerd.io

# dnf repository-packages '*docker*' list

sudo rm -rf /etc/yum.repos.d/docker-ce.repo

# Remove all container, images, volumes, ...
```

# podman (rootless)

## Install

```shell
sudo dnf install podman podman-compose
```

## Enable user podman socket

```shell
systemctl --user enable --now podman.socket
```
