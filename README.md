# Xiaomi 11T Pro (vili) — Root & Docker Lab

Turning a Xiaomi 11T Pro into a portable Docker/container lab.

| | |
|---|---|
| **Device** | Xiaomi 11T Pro (`vili`) |
| **Stock OS** | MIUI 14.0.12.0 (TKDEUXM) / Android 13 |
| **Goal** | Custom AOSP ROM + Docker-capable kernel |

---

## Paths

**Path 1 & 2 — Root only (no Docker)**
Unlock bootloader → patch `boot.img` with Magisk → flash. Documented in [docs/PROCESS.md](docs/PROCESS.md).
Docker is not achievable on this path. The stock MIUI kernel lacks the required container primitives and cannot be replaced without bootlooping.

**Path 3 — Docker lab ⭐**
Flash a custom AOSP ROM (crDroid recommended — official vili support, weekly builds) + a Docker-capable kernel, then install Docker via Termux. Full guide: [docs/DOCKER.md](docs/DOCKER.md).

---

## Repository Contents

| Path | Description |
|------|-------------|
| `boot_images/boot.img` | Stock boot image (MIUI 14 fastboot ROM) |
| `boot_images/magisk_patched.img` | Magisk-patched boot image (Path 1/2) |
| `boot_images/vbmeta.img` | Stock vbmeta (disable AVB if needed) |
| `docs/PROCESS.md` | Bootloader unlock → Magisk root walkthrough |
| `docs/DOCKER.md` | Path 3: full Docker lab guide |
| `docs/Versions.txt` | Firmware version reference |
| `.github/workflows/build-kernel.yml` | GitHub Actions workflow to compile a Docker-capable kernel |

> `boot.img` and `magisk_patched.img` are stored via Git LFS (~200 MB each).
