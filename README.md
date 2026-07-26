# Xiaomi 11T Pro (vili) - Root & Docker Kernel

This repository contains the necessary files to root the Xiaomi 11T Pro and track the progress of building a custom kernel with native Docker support.

## 📱 Device Specifications
* **Device:** Xiaomi 11T Pro
* **Codename:** `vili`
* **OS Version:** MIUI 14.0.12.0 (TKDEUXM)
* **Android Version:** 13 (TKQ1.220829.002)

## 📂 Repository Contents
* `boot_images/boot.img`: The stock boot image extracted from the official fastboot ROM.
* `boot_images/magisk_patched.img`: The boot image patched via Magisk for root access. *(Note: These files are managed via Git LFS).*
* `docs/Versions.txt`: Detailed version tracking.

## 🚀 How to Flash Root
Ensure you have `adb` and `fastboot` installed on your machine.

1. Reboot the device into Fastboot mode (hold **Volume Down + Power**).
2. Connect the device to your computer via USB.
3. Open your terminal in the `boot_images` directory and run:
   ```bash
   fastboot flash boot magisk_patched.img
   fastboot reboot
