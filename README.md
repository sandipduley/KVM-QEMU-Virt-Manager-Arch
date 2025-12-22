# KVM, QEMU & Virt-Manager Setup on Arch Linux (Hyprland)

## Overview

This document provides a **professional, Arch-native guide** to setting up **KVM, QEMU, libvirt, and virt-manager** on **Arch Linux running Hyprland (Wayland)**. The instructions avoid desktop-environment assumptions (GNOME/KDE) and are tailored for lightweight Wayland compositors.

---

## 1. Prerequisites

### 1.1 Enable Virtualization in BIOS/UEFI

Ensure virtualization is enabled before proceeding:

- **Intel CPUs**: Enable **VT-x** and **VT-d**
- **AMD CPUs**: Enable **SVM** and **IOMMU**

Verify from Arch Linux:

```bash
lscpu | grep Virtualization
```

Expected output:

- `VT-x` (Intel)
- `AMD-V` (AMD)

---

## 2. Install Required Packages

Install core virtualization components:

```bash
sudo pacman -S --needed \
  qemu-full \
  virt-manager \
  libvirt \
  dnsmasq \
  bridge-utils \
  vde2 \
  ebtables \
  iptables-nft \
  openbsd-netcat
```

Recommended utilities:

```bash
sudo pacman -S virt-viewer spice spice-gtk
```

---

## 3. Enable and Start libvirt

Enable the libvirt daemon:

```bash
sudo systemctl enable libvirtd.service
sudo systemctl start libvirtd.service
```

Verify status:

```bash
systemctl status libvirtd
```

---

## 4. Configure User Permissions

Add your user to the `libvirt` group to avoid repeated sudo usage:

```bash
sudo usermod -aG libvirt $(whoami)
```

Apply group changes:

```bash
newgrp libvirt
```

(Alternatively, log out and log back in.)

---

## 5. Enable Default Virtual Network (NAT)

Start and autostart the default NAT network:

```bash
sudo virsh net-autostart default
sudo virsh net-start default
```

Verify:

```bash
virsh net-list --all
```

Expected:

```
default   active   yes  --  If no response then also it will work
```

---

## 6. Verify KVM Kernel Modules

Check loaded modules:

```bash
lsmod | grep kvm
```

Expected modules:

- `kvm_intel` (Intel)
- `kvm_amd` (AMD)

If not loaded:

```bash
sudo modprobe kvm_intel
# or
sudo modprobe kvm_amd
```

---

## 7. libvirt Socket Permissions (Important for Hyprland)

Edit the libvirt configuration:

```bash
sudo nvim /etc/libvirt/libvirtd.conf
```

Uncomment and set:

```conf
unix_sock_group = "libvirt"
unix_sock_rw_perms = "0770"
```

Restart libvirt:

```bash
sudo systemctl restart libvirtd
```

---

## 8. Wayland & Hyprland Considerations

Virt-manager works natively on Wayland. For best compatibility:

```bash
sudo pacman -S gtk3 gtk4
```

Use **SPICE** and **QXL** for VM display to avoid black screens.

---

## 9. Launch virt-manager

From terminal:

```bash
virt-manager
```

Or via a `.desktop` launcher:

```bash
/usr/bin/virt-manager
```

---

## 10. Recommended VM Configuration

### Firmware & Chipset

- Firmware: **UEFI (OVMF)**
- Chipset: **Q35**

### CPU

- Mode: **host-passthrough**

### Display

- Display Server: **SPICE**
- Video Device: **QXL**

### Storage

- Disk Bus: **VirtIO**

### Network

- Device Model: **virtio**

---

## 11. Windows Guest Requirements

For Windows virtual machines:

- Download `virtio-win.iso`
- Attach it as a secondary CD-ROM
- Install drivers during OS installation

---

## 12. Performance Optimization (Optional)

### Enable HugePages

Create configuration file:

```bash
sudo nvim /etc/sysctl.d/40-hugepage.conf
```

Add:

```conf
vm.nr_hugepages=1024
```

Apply:

```bash
sudo sysctl -p
```

---

## 13. Troubleshooting

### Issue: “QEMU/KVM not available”

```bash
sudo pacman -S linux-headers
reboot
```

---

### Issue: Permission denied on libvirt socket

```bash
groups | grep libvirt
```

Ensure your user belongs to the `libvirt` group and re-login if needed.

---

### Issue: VM Window Opens but Display is Blank

- Switch display to **SPICE**
- Set video to **QXL**
- Disable **OpenGL** in VM settings

---

## 14. Verification

Confirm VM visibility:

```bash
virsh list --all
```

If virtual machines are listed, the setup is complete.

---

## 15. Advanced Use Cases (Optional)

- Nested virtualization (e.g., Kali inside Kali)
- PCI / GPU passthrough
- Isolated malware analysis labs
- Snapshot-based exploit testing

---

## Conclusion

You now have a **stable, professional KVM/QEMU virtualization environment** on **Arch Linux with Hyprland**. This setup is suitable for development, testing, cybersecurity labs, and advanced virtualization workflows.

---
