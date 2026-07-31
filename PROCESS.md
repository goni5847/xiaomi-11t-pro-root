# Xiaomi 11T Pro (vili) — Rooting & Prep Journey

This document logs the exact step-by-step process used to unlock the bootloader, extract and patch the boot image, set up Git LFS, and flash root on the **Xiaomi 11T Pro**.

## Device Specifications
* **Device:** Xiaomi 11T Pro
* **Codename:** `vili`
* **OS Version:** MIUI 14.0.12.0 (TKDEUXM)
* **Android Version:** 13 (TKQ1.220829.002)

> ⚠️ **Before you start:** Unlocking the bootloader wipes the device completely. Back up anything you care about (photos, app data, etc.) before beginning Phase 1.

---

## Phase 1: Bootloader Unlock (Windows)

This was done on Windows, since the Xiaomi bootloader unlock tool is only available for that platform.

### 1. Enable Developer Options & OEM Unlocking

* Go to **Settings > About Phone** and tap **MIUI Version** seven times to enable Developer Options.
* Go to **Settings > Additional Settings > Developer Options** and enable **OEM unlocking**.
  * This step is required before the Mi Unlock Tool can do anything — without it, the unlock will fail even after the waiting period.

### 2. Account Binding & The Waiting Period

To unlock a Xiaomi device, the Mi Account must be bound to the phone's hardware.

**Important: use the same Mi Account throughout the entire process, and make sure of this at every step.**

* Turn off Wi-Fi and use Mobile Data instead.
* Navigate to **Settings > Additional Settings > Developer Options > Mi Unlock Status**.
* Tap **Add account and device**.

**Crucial rule:** After this step, a mandatory **168-hour (7-day)** waiting period begins. During this time:
* Do **not** log out of the Mi Account.
* Do **not** factory reset the device.
* Do **not** click "Add account" again — any of these actions resets the 7-day timer.

In practice, the simplest approach is to leave the phone aside, powered off, for the full 7 days. You can remove the SIM card if you like, just remember to put it back afterward.

### 3. Unlocking with the Mi Unlock Tool

Once the 168 hours have passed:

* Boot the phone into **Fastboot Mode** (hold `Volume Down + Power`).
* Connect the phone to the Windows PC via USB.
* Open the Mi Unlock Tool, log in with the same Mi Account, and run the unlock process.
* The tool wipes the device (factory reset) and the bootloader is officially unlocked.

**Troubleshooting — "Not Connected" error:**
The PC initially failed to recognize the phone in Fastboot mode. This was resolved by installing the correct **Android Bootloader Interface** drivers via the gear icon (Settings) in the Mi Unlock Tool.

---

## Phase 2: Firmware & Magisk Patching

### 1. Extracting the Stock Boot Image

* Download the exact matching **Fastboot ROM** (`.tgz` format — *not* the `.zip` recovery format) for MIUI 14.0.12.0 (TKDEUXM). This version string is usually listed in the phone's detailed specs or in **Settings > About Phone**.
* Extract the archive and locate `boot.img` inside the `images` folder.
* Transfer `boot.img` to the phone's internal storage (the `Download` folder is fine, for easy access).

### 2. Patching with Magisk

* Download and install the official Magisk APK from [topjohnwu's GitHub](https://github.com/topjohnwu/Magisk) on the phone.
* Open Magisk > **Install** > **Select and Patch a File**.
* Select the stock `boot.img`. Magisk processes it and generates a new file named `magisk_patched.img` in the `Download` folder.
* Transfer this patched image back to the PC. If the patched file doesn't appear in the directory it should, copy the file in the phone to root/internal storage folder.

---

## Phase 3: Repository Setup & Git LFS (Windows)

Because `boot.img` and `magisk_patched.img` are each roughly 196 MB, they exceed GitHub's 100 MB file size limit. **Git LFS (Large File Storage)** solves this by storing large files outside the normal Git history.

### 1. Initialize the Repository and Git LFS

Open Git Bash in the project folder and run:

```bash
git init
git lfs install
git lfs track "*.img"
git add .gitattributes
```

### 2. Add, Commit, and Push the Files

```bash
git add boot.img magisk_patched.img
git commit -m "Add stock and Magisk-patched boot images"
git remote add origin <your-repository-url>
git push -u origin main
```

**Notes:**
* The first push will take noticeably longer than usual, since Git LFS uploads the full binary content of both `.img` files (~400 MB combined).
* Run `git lfs ls-files` at any point to confirm the images are being tracked by LFS rather than committed directly to Git history.
* If you ever clone this repo elsewhere, run `git lfs install` on that machine too, or the `.img` files will download as small pointer files instead of the real content.

---

## Phase 4: Flashing the Patched Boot Image

With `magisk_patched.img` on the PC and the phone in Fastboot mode:

### 1. Confirm the Device Is Detected

```bash
fastboot devices
```

You should see your device's serial number listed. If nothing appears, revisit the driver fix from Phase 1.3.

### 2. Flash the Patched Boot Image

```bash
fastboot flash boot magisk_patched.img
```

### 3. Reboot

```bash
fastboot reboot
```

The phone should boot normally into MIUI, and Magisk should show as installed with root granted.

### ⚠️ Troubleshooting: Bootloop after flashing

Some Xiaomi 11T Pro (`vili`) firmware builds enforce strict dm-verity / AVB (Android Verified Boot) checks, which can cause a bootloop even with a correctly patched boot image. If this happens:

1. Boot back into Fastboot mode.
2. Extract `vbmeta.img` from the same fastboot ROM used in Phase 2.
3. Flash it with verification disabled:

```bash
fastboot --disable-verity --disable-verification flash vbmeta vbmeta.img
```

4. Reboot again with `fastboot reboot`.

This disables AVB/dm-verity enforcement, which commonly resolves boot failures caused by the modified boot partition.

---

## Summary

| Phase | Goal | Platform |
|---|---|---|
| 1 | Unlock bootloader (incl. 7-day wait) | Windows + Phone |
| 2 | Extract stock boot.img, patch with Magisk | Phone |
| 3 | Store large `.img` files with Git LFS | Windows |
| 4 | Flash patched boot (+ vbmeta if needed) | Windows |