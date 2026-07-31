# Xiaomi 11T Pro (vili) — Bootloader Unlock & Magisk Root

Step-by-step log for unlocking the bootloader and rooting with Magisk (Path 1 & 2).
For Docker support, this is a prerequisite — then continue with [DOCKER.md](DOCKER.md).

> ⚠️ Unlocking the bootloader wipes the device completely. Back up everything before starting.

---

## Phase 1: Bootloader Unlock (Windows only)

The Xiaomi bootloader unlock tool is Windows-only.

### 1.1 Enable Developer Options & OEM Unlocking

- Go to **Settings > About Phone**, tap **MIUI Version** seven times to enable Developer Options.
- Go to **Settings > Additional Settings > Developer Options**, enable **OEM unlocking**.

### 1.2 Account Binding & the 7-day Wait

- Turn off Wi-Fi, use mobile data.
- Go to **Settings > Additional Settings > Developer Options > Mi Unlock Status**.
- Tap **Add account and device**.

After this, a mandatory **168-hour (7-day)** waiting period begins. During this time:
- Do not log out of the Mi Account.
- Do not factory reset the device.
- Do not tap "Add account" again.

Any of those actions resets the timer. Simplest approach: power the phone off and leave it for 7 days.

### 1.3 Run the Mi Unlock Tool

1. Boot into Fastboot mode: hold **Volume Down + Power**.
2. Connect to PC via USB.
3. Open Mi Unlock Tool, log in with the same Mi Account, run the unlock.
4. The tool wipes the device — bootloader is now unlocked.

**Troubleshooting — "Not Connected" error:** Install the Android Bootloader Interface drivers via the gear icon (Settings) inside the Mi Unlock Tool.

---

## Phase 2: Patch Boot Image with Magisk

### 2.1 Get the stock boot image

- Download the **Fastboot ROM** (`.tgz` format, not `.zip`) matching your exact build: MIUI 14.0.12.0 (TKDEUXM).
- Extract it and locate `boot.img` inside the `images/` folder.
- Transfer `boot.img` to the phone's internal storage.

### 2.2 Patch with Magisk

- Install the Magisk APK from [topjohnwu's GitHub](https://github.com/topjohnwu/Magisk).
- Open Magisk → **Install** → **Select and Patch a File** → select `boot.img`.
- Magisk writes `magisk_patched.img` to the Downloads folder.
- Transfer it back to your PC.

---

## Phase 3: Flash

```bash
# Confirm device is detected
fastboot devices

# Flash the patched image
fastboot flash boot magisk_patched.img
fastboot reboot
```

The phone boots into MIUI with Magisk installed and root active.

### Troubleshooting: bootloop after flashing

If the device bootloops, AVB/dm-verity is rejecting the modified partition. Fix:

```bash
fastboot --disable-verity --disable-verification flash vbmeta vbmeta.img
fastboot reboot
```

`vbmeta.img` is included in this repo (`boot_images/vbmeta.img`) and in the fastboot ROM.

---

## Summary

| Phase | Goal |
|-------|------|
| 1 | Unlock bootloader (7-day wait required) |
| 2 | Patch stock `boot.img` with Magisk |
| 3 | Flash patched image via fastboot |

Result: rooted MIUI 14. Docker is not possible on this base — see [DOCKER.md](DOCKER.md) for Path 3.
