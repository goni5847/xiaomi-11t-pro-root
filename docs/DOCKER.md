# Path 3: Docker Lab on Xiaomi 11T Pro (vili)

Running Docker on this device requires a kernel with cgroups, namespaces, and overlayfs — none of which are guaranteed in the stock MIUI kernel, and the MIUI kernel cannot be replaced without bootlooping (Xiaomi never published Android 13 kernel source for vili). The only path is a custom AOSP ROM paired with a Docker-capable kernel.

**Prerequisites:** bootloader unlocked ([PROCESS.md](PROCESS.md) Phase 1).

---

## Step 1: Pick a Custom ROM

| ROM | Status for vili | Notes |
|-----|----------------|-------|
| **crDroid** | Official, actively maintained | Recommended — weekly builds, maintained by swiitchOFF who also maintains the primary vili kernel |
| **LineageOS** | Unofficial XDA build only | No official support, no wiki page, no guaranteed updates |
| **PixelOS** | Community build | Stock Pixel feel |
| **DerpFest** | Community build | Feature-rich alternative |

**Use crDroid.** It has official vili support (v12, Android 16, weekly builds) and is maintained by swiitchOFF — the same person who maintains the SwithOFF kernel. ROM and kernel from the same maintainer means a guaranteed matched pair, which is exactly what you need.

Download: https://crdroid.net/vili/12
Recovery (OrangeFox for vili): linked on the crDroid download page

---

## Step 2: Flash the Custom ROM

You need: the ROM zip, a custom recovery (OrangeFox or TWRP for vili), and optionally GApps.

### Flash the custom recovery

```bash
fastboot devices                          # confirm device detected
fastboot flash recovery recovery.img
fastboot reboot recovery
```

### Wipe from recovery

1. **Wipe > Format Data** — full wipe, all user data gone.
2. **Wipe > Advanced Wipe** — wipe Dalvik/ART cache.

### Flash the ROM

```bash
adb sideload rom.zip
```

Flash GApps immediately after, before rebooting. First boot takes 3–5 minutes — do not interrupt it.

---

## Step 3: Flash a Docker-capable Kernel

Community vili kernels (all AOSP only):

| Kernel | Notes |
|--------|-------|
| **SwithOFF kernel** | Primary choice — same maintainer as crDroid vili, tested together |
| TUF kernel | Performance-focused |
| Reborn kernel | Feature-rich |
| Team Vili Sector5 | Community build |

If you're on crDroid, start with the SwithOFF kernel — it's built and tested against the same crDroid base.

Before flashing, check the kernel's posted config for these flags:

```
CONFIG_NAMESPACES=y
CONFIG_USER_NS=y
CONFIG_NET_NS=y
CONFIG_CGROUPS=y
CONFIG_MEMCG=y
CONFIG_OVERLAY_FS=y
```

All six must be present for Docker to work. If none of the community kernels have them, try a different kernel build.

Most community kernels ship as a flashable zip — sideload from recovery:

```bash
adb sideload kernel.zip
```

---

## Step 4: Root with Magisk

crDroid ships as an OTA zip containing `payload.bin` — there is no flat `boot.img` to extract directly. You need to dump it first.

### Extract boot.img from payload.bin

```bash
pip install payload_dumper
payload_dumper --partitions boot payload.bin
# outputs boot.img to ./output/
```

### Patch with Magisk

Transfer `boot.img` to the phone:
```bash
adb push output/boot.img /sdcard/Download/boot.img
```

Open Magisk → **Install** → **Select and Patch a File** → select `boot.img`. Magisk writes `magisk_patched_XXXXX.img` to `/sdcard/Download/`.

Pull it back to your PC:
```bash
adb pull /sdcard/Download/magisk_patched_XXXXX.img boot_images/magisk_patched_cdDroid.img
```

### Flash

```bash
# from the repo root
adb reboot bootloader
fastboot flash boot boot_images/magisk_patched_cdDroid.img
fastboot reboot
```

After first boot, open Magisk. If it shows "Requires Additional Setup", complete it and reboot. Then in Termux:

```bash
su
whoami   # should return: root
```

Magisk will show a popup to grant root to Termux — tap **Grant**. Root confirmed working with Magisk 30700 on crDroid 12.11.

---

## Step 5: Install Docker via Termux

Install Termux from **F-Droid** (not the Play Store — that version is outdated):
https://f-droid.org/packages/com.termux/

Grant Termux root access in Magisk → SuperUser.

```bash
# In Termux
pkg update && pkg upgrade
pkg install root-repo
pkg install docker
```

Start the daemon and verify:

```bash
su
dockerd &
# wait ~15 seconds
docker info
docker run hello-world
```

Verify kernel support against Docker's requirements:

```bash
pkg install curl
curl -fsSL https://raw.githubusercontent.com/moby/moby/master/contrib/check-config.sh | bash
```

All `Generally Necessary` items should show `enabled`. Misses in `Optional Features` are acceptable.

---

## Troubleshooting

**Docker daemon fails to start**
Run `dmesg | grep -i cgroup`. If cgroup errors appear, the kernel is missing required flags. Try a different community kernel. Ensure you ran `su` before `dockerd`.

**`docker run` fails with network errors**
The kernel needs `CONFIG_VETH`, `CONFIG_BRIDGE`, and `CONFIG_IP_NF_TARGET_MASQUERADE`. Check the kernel config.

**Bootloop after kernel flash**
The kernel and ROM are incompatible. Boot into recovery, flash the ROM's stock kernel, and try a different kernel build.

**`check-config.sh` reports missing cgroup features**
Some Docker features require cgroup v2 (`CONFIG_CGROUP_V2=y`). Check the kernel's XDA thread — most community kernels document their cgroup support level.
