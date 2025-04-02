# Build Guide: BlueZ-ALSA Utils for Volumio (Multi-Arch)

This guide describes how to build **`bluez-alsa-utils` v4.3.1** for **Volumio OS** using Docker and QEMU. It supports builds for `armhf`, `arm64`, `amd64`, and `armv6` targets from an Ubuntu 24.04 host.

---

## 🔧 Host Requirements

- **OS**: Ubuntu 24.04 (or compatible)
- **Docker**: Install via `sudo apt install docker.io`
- **QEMU Multi-Arch Support**:
  ```bash
  docker run --rm --privileged multiarch/qemu-user-static --reset -p yes
  ```

---

## 📁 Project Layout

```
volumio-bluetooth-core/
├── build/
│   └── bluez-alsa-utils/
│       ├── source/         # Extracted source
│       ├── debian/         # Debian packaging files
│       └── patches/        # Optional patches
├── docker/
│   ├── Dockerfile.alsa.*   # One per architecture
│   └── run-docker-alsa-utils.sh
├── out/                    # Architecture-specific .deb output
├── package-sources/        # Original tarballs and packaging
├── scripts/
│   └── extract-bluez-alsa-utils-source.sh
└── README.md
```

---

## 🧭 Build Flow Overview


| Environment    | Role                             |
| ---------------- | ---------------------------------- |
| **Host**       | Runs Docker builds, edits files  |
| **Docker**     | Cross-compiles .deb packages     |
| **Volumio OS** | Installs resulting .deb packages |

---

## 🧱 Step-by-Step Build Instructions

### 1. Prepare Source & Packaging

Run the extraction script to set up `bluez-alsa-utils` source and Debian metadata:

```bash
./scripts/extract-bluez-alsa-utils-source.sh
```

This script:

- Extracts **bluez-alsa v4.3.1** source
- Applies Volumio-specific packaging metadata

---

### 2. Update the Debian Changelog

Edit the changelog to reflect a Volumio-custom build:

```bash
nano build/bluez-alsa-utils/debian/changelog
```

Insert a new entry at the top:

```
bluez-alsa-utils (4.3.1-1volumio1) bookworm; urgency=medium

  * Volumio-customized bluez-alsa-utils build

 -- Your Name <you@example.com>  Tue, 26 Mar 2024 12:00:00 +0000
```

---

### 3. Build for Target Architectures

Use the build script to generate `.deb` packages:

```bash
# ARMv7 (armhf)
./docker/run-docker-alsa-utils.sh bluez-alsa-utils armhf --verbose

# ARMv8 (arm64)
./docker/run-docker-alsa-utils.sh bluez-alsa-utils arm64 --verbose

# x86_64 (amd64)
./docker/run-docker-alsa-utils.sh bluez-alsa-utils amd64 --verbose

# ARMv6 (universal Pi target using armhf ABI)
./docker/run-docker-alsa-utils.sh bluez-alsa-utils armv6 --verbose
```

Output files appear in:

```
out/armhf/bluez-alsa-utils_4.3.1-1volumio1_armv7.deb
out/arm64/bluez-alsa-utils_4.3.1-1volumio1_armv8.deb
out/amd64/bluez-alsa-utils_4.3.1-1volumio1_x64.deb
out/armv6/bluez-alsa-utils_4.3.1-1volumio1_arm.deb
```

> Note: ARMv6 builds use `armhf` ABI with `-march=armv6 -mfloat-abi=hard -mfpu=vfp -marm` and are compatible with all Pi models (Pi 0–5).

---

## 📦 Installing on Volumio OS

Transfer and install the built packages on your Volumio system:

```bash
dpkg -i out/armv6/bluez-alsa-utils_4.3.1-1volumio1_arm.deb
dpkg -i out/armv6/bluez-alsa-utils-common_4.3.1-1volumio1_all.deb

# Prevent accidental upgrades
apt-mark hold bluez-alsa-utils bluez-alsa-utils-common
```

> Replace `armv6` with `armhf`, `arm64`, or `amd64` depending on the target.

---

## 🔁 Building for Multiple Architectures

You can repeat the build process across architectures using:

```bash
./docker/run-docker-alsa-utils.sh bluez-alsa-utils <arch> volumio --verbose
```

Example:

```bash
./docker/run-docker-alsa-utils.sh bluez-alsa-utils arm64 volumio --verbose
```

---

## 💡 Notes

- Use the correct Dockerfile per architecture:
  - `Dockerfile.alsa.armv6`, `Dockerfile.alsa.armhf`, etc.
- Re-run the extraction script if you want to rebase on a new upstream version.
- `--verbose` adds build progress logs for debugging or CI diagnostics.
