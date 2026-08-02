# Post-Root Setup

Steps to configure the device after rooting, regardless of path taken.

---

## Network Setup (Static IP)

To give the device a stable IP address on the local network:

**1. Disable MAC address randomisation on the phone**
- Go to **Settings > Wi-Fi** → tap your network → **Privacy** → set to **Use device MAC**
- This ensures the router always sees the same MAC address

**2. Create a static DHCP lease on your router**
- Log into your router admin panel
- Find the DHCP lease table and assign a fixed IP to the device's MAC address
- The device will now always get the same IP, even after reboots

This is required for SSH and code-server to be reachable at a consistent address.

---

## Termux Auto-Start Services (Termux:Boot)

Two services are configured to start automatically on boot using **Termux:Boot** (installed from F-Droid).

**Install from F-Droid:**
- [Termux](https://f-droid.org/packages/com.termux/)
- [Termux:Boot](https://f-droid.org/packages/com.termux.boot/)

> NOTE: Do not install Termux or Termux:Boot from the Play Store — those versions are outdated and incompatible with F-Droid packages.

Scripts in `~/.termux/boot/` are executed in alphabetical order on device boot. The prefix number controls execution order.

### ~/.termux/boot/start-1sshd

Starts the SSH daemon and acquires a wake lock to keep the device awake.

```sh
#!/data/data/com.termux/files/usr/bin/sh
termux-wake-lock
sshd
```

- `termux-wake-lock` prevents Android from killing Termux in the background
- `sshd` starts the SSH server (default port 8022)

### ~/.termux/boot/start-2code-server

Starts code-server (VS Code in the browser).

```sh
#!/data/data/com.termux/files/usr/bin/sh
pkill code-server
code-server
```

- `pkill code-server` ensures no stale instance is running before starting
- code-server config is at `~/.config/code-server/config.yaml`

### code-server config

```yaml
bind-addr: 0.0.0.0:8080
auth: password
password: <your-password>
cert: false
```

`bind-addr: 0.0.0.0` is required — the default `127.0.0.1` only allows connections from the device itself, blocking access from your PC.

Access code-server from any device on the local network at `http://<device-ip>:8080`.

### Deploying scripts to a new device

```bash
adb push termux-boot/start-1sshd /sdcard/Download/
adb push termux-boot/start-2code-server /sdcard/Download/
```

Then in Termux on the device:
```bash
mkdir -p ~/.termux/boot
cp /sdcard/Download/start-1sshd ~/.termux/boot/
cp /sdcard/Download/start-2code-server ~/.termux/boot/
chmod +x ~/.termux/boot/*
```
