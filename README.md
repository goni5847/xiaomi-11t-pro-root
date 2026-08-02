# Xiaomi 11T Pro (vili) — Root & Docker Lab

Turning a Xiaomi 11T Pro into a portable Docker/container lab.

| | |
|---|---|
| **Device** | Xiaomi 11T Pro (`vili`) |
| **Stock OS** | MIUI 14.0.12.0 (TKDEUXM) / Android 13 |
| **Current OS** | crDroid 12.11 (Android 16) |
| **Root** | Magisk 30700 |
| **Goal** | Custom AOSP ROM + Docker-capable kernel |

---

## Paths

**Path 1 & 2 — Root only (no Docker)**
Unlock bootloader → patch `boot.img` with Magisk → flash. Documented in [docs/PROCESS.md](docs/PROCESS.md).
Docker is not achievable on this path. The stock MIUI kernel lacks the required container primitives and cannot be replaced without bootlooping.

**Path 3 — Docker lab (recommended)**
Flash a custom AOSP ROM (crDroid recommended — official vili support, weekly builds) + a Docker-capable kernel, then install Docker via Termux. Full guide: [docs/DOCKER.md](docs/DOCKER.md).

---

## Repository Contents

| Path | Description |
|------|-------------|
| `boot_images/crDroidBoot.img` | Stock boot image extracted from crDroid 12.11 `payload.bin` |
| `boot_images/magisk_patched_cdDroid.img` | Magisk-patched boot image (crDroid 12.11) — confirmed rooted |
| `boot_images/OrangeFox-R12.0-v10-vili.img` | OrangeFox recovery for vili |
| `boot_images/vbmeta.img` | Stock vbmeta (disable AVB if needed) |
| `termux-boot/start-1sshd` | Auto-start script: wake lock + sshd |
| `termux-boot/start-2code-server` | Auto-start script: code-server |
| `docs/PROCESS.md` | Bootloader unlock → Magisk root walkthrough |
| `docs/DOCKER.md` | Path 3: full Docker lab guide |
| `docs/SETUP.md` | Post-root setup: network, Termux auto-start, code-server |
| `docs/Versions.txt` | Firmware version reference |
| `.github/workflows/build-kernel.yml` | GitHub Actions workflow to compile a Docker-capable kernel |

> Boot images are stored via Git LFS (~200 MB each).
