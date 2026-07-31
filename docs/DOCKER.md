# Path 3: Docker Lab on Xiaomi 11T Pro (vili)

Running Docker on this device requires a kernel with cgroups, namespaces, and overlayfs — none of which are guaranteed in the stock MIUI kernel, and the MIUI kernel cannot be replaced without bootlooping (Xiaomi never published Android 13 kernel source for vili). The only path is a custom AOSP ROM paired with a Docker-capable kernel.

**Prerequisites:** bootloader unlocked ([PROCESS.md](PROCESS.md) Phase 1).

---

## Step 1: Pick a Custom ROM

| ROM | Status for vili | Notes |
|-----|----------------|-------|
| **crDroid** | ✅ Official, actively maintained | Recommended — weekly builds, maintained by swiitchOFF who also maintains the primary vili kernel |
| **LineageOS** | ⚠️ Unofficial XDA build only | No official support, no wiki page, no guaranteed updates |
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

All six must be present for Docker to work. If none of the community kernels have them, skip to Step 6 to build your own.

Most community kernels ship as a flashable zip — sideload from recovery:

```bash
adb sideload kernel.zip
```

---

## Step 4: Root with Magisk

Extract `boot.img` from the ROM zip, patch it with the Magisk app (same process as [PROCESS.md](PROCESS.md) Phase 2), then flash:

```bash
fastboot flash boot magisk_patched.img
fastboot reboot
```

Some ROMs support flashing Magisk as a zip directly from recovery — check the ROM's own documentation.

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

## Step 6 (Optional): Build Your Own Kernel

If no community kernel has all required flags, use the `build-kernel.yml` workflow in this repo. It clones the kernel source, injects all Docker config flags, compiles with Proton Clang, and uploads the `Image` as an artifact.

1. Go to **Actions > Build Xiaomi 11T Pro (vili) Kernel > Run workflow**.
2. Set `KERNEL_SOURCE`, `KERNEL_BRANCH`, and `DEFCONFIG` for your chosen source.
3. Download the `Compiled_Kernel_Image` artifact.
4. Package with AnyKernel3 and flash from recovery, or inject into `boot.img` and flash via fastboot.

This kernel must be paired with an AOSP ROM — not MIUI. See the workflow file's warning header for details.

---

## Troubleshooting

**Docker daemon fails to start**
Run `dmesg | grep -i cgroup`. If cgroup errors appear, the kernel is missing required flags. Try a different community kernel or build your own (Step 6). Ensure you ran `su` before `dockerd`.

**`docker run` fails with network errors**
The kernel needs `CONFIG_VETH`, `CONFIG_BRIDGE`, and `CONFIG_IP_NF_TARGET_MASQUERADE`. Check the kernel config.

**Bootloop after kernel flash**
The kernel and ROM are incompatible. Boot into recovery, flash the ROM's stock kernel, and try a different kernel build.

**`check-config.sh` reports missing cgroup features**
Some Docker features require cgroup v2 (`CONFIG_CGROUP_V2=y`). Check the kernel's XDA thread — most community kernels document their cgroup support level.
