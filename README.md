🌐 **English** | [Português](./README.pt-BR.md)

# 🔆 Linux Backlight Fix — Hybrid Graphics Laptops (Intel/AMD + NVIDIA)

> Fix brightness controls (Fn keys + system sliders) on Linux for laptops with hybrid graphics.

Common on gaming and productivity laptops from Acer, ASUS, Lenovo, Dell, MSI, and others — especially those with Intel/AMD integrated GPU + NVIDIA discrete GPU.

---

## The problem

After installing Linux, the brightness keys (Fn+F5/F6 or similar) and system brightness sliders stop working. The screen gets stuck at full brightness or won't respond at all.

**Why it happens:** On hybrid graphics laptops, the kernel and the firmware (BIOS/UEFI) often fight over who controls the backlight interface. The `acpi_backlight` kernel parameter lets you explicitly pick who wins.

---

## Fix

Edit `/etc/default/grub` and find:

```
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"
```

Add the parameter that works for your setup (see options below). Example:

```
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash acpi_backlight=native"
```

Then regenerate the GRUB config:

```bash
# Ubuntu / Pop!_OS / Debian / Mint
sudo update-grub

# Arch / Manjaro / EndeavourOS
sudo grub-mkconfig -o /boot/grub/grub.cfg

# Fedora / RHEL
sudo grub2-mkconfig -o /boot/grub2/grub.cfg
```

Reboot and test your brightness keys.

---

## Which parameter should I use?

Try them in this order until one works:

### ✅ `acpi_backlight=native` — try this first
```
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash acpi_backlight=native"
```
Tells the kernel to use the hardware's native ACPI backlight directly, bypassing firmware. **Works on most modern hybrid laptops** (Intel 10th gen+, Ryzen 5000+).

---

### `acpi_backlight=vendor`
```
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash acpi_backlight=vendor"
```
Uses the laptop vendor's ACPI implementation instead of the generic one. Try this if `native` doesn't work — common on older models or specific OEM firmware.

---

### `acpi_backlight=video`
```
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash acpi_backlight=video"
```
Delegates backlight control to the GPU driver. Can work well on setups where the Intel/AMD iGPU driver handles display output.

---

### `acpi_backlight=none` + `nvidia_drm.modeset=1` — proprietary NVIDIA driver
```
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash acpi_backlight=none nvidia_drm.modeset=1"
```
Disables ACPI backlight and lets the proprietary NVIDIA modesetting driver take over. Try this if you're using the official NVIDIA driver (not nouveau) and none of the above work.

---

### `video.use_native_backlight=1` — older kernels (< 5.x)
```
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash video.use_native_backlight=1"
```
An older parameter with similar effect to `native`. Useful on kernels before 5.x or on distros that haven't caught up yet.

---

## Tested configurations

| Model | CPU | GPU | Distro | Kernel | Fix |
|---|---|---|---|---|---|
| Acer Nitro AN515-58 | Intel i5-12500H | RTX 3050 | Ubuntu 22.04, Pop!_OS, Arch | 5.15 – 6.x | `acpi_backlight=native` ✅ |

**Did it work on your machine?** Open a PR or issue and I'll add it to the table!

---

## Still not working?

- **Secure Boot enabled?** Some systems ignore kernel parameters when Secure Boot is on. Disable it in BIOS and try again.
- **Using Wayland?** Try switching to an X11 session first to rule out compositor issues.
- **Using NVIDIA proprietary driver?** Don't mix `acpi_backlight=native` with `nvidia_drm.modeset=1` — use the dedicated combo above instead.
- **Check what your system reports:**
  ```bash
  ls /sys/class/backlight/
  cat /sys/class/backlight/*/brightness
  ```
  If the directory is empty, try `acpi_backlight=vendor` or `none`.

---

## Contributing

If you got it working on a laptop not listed above, please open an issue or PR with:

- Laptop model
- CPU and GPU
- Distro and kernel version (`uname -r`)
- The parameter that worked

---

## References

- [kernel.org — kernel parameters](https://www.kernel.org/doc/html/latest/admin-guide/kernel-parameters.html)
- [ArchWiki — Backlight](https://wiki.archlinux.org/title/Backlight)
- [ArchWiki — NVIDIA DRM kernel mode setting](https://wiki.archlinux.org/title/NVIDIA#DRM_kernel_mode_setting)
