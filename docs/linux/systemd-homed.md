systemd-homed
===

***Some systemd-homed / homectl related information***

**Author**: *Markus Rathgeb*

---

# Fedora

1. Enabling systemd-homed (it should be enabled but still)
   ```
   sudo systemctl enable systemd-homed.service
   ```
1. Enabling systemd-home PAM
   ```
   sudo authselect enable-feature with-systemd-homed
   ```
1. Creating a homed-managed user with homectl. 
   You can refer to homectl documentation to explore various options, e.g., adding FIDO2 with Yubikey, selecting filesystem type, setting home dir’s size, adding homed user to the wheel or any other group, etc.

Examples

```
sudo \
    homectl \
        --image-path=/dev/sda \
        --storage=luks \
        --fs-type=btrfs \
        create username
```