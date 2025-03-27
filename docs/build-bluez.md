# Build Guide: BlueZ for Volumio (Multi-Arch)

This guide outlines how to build BlueZ 5.72 for Volumio using Docker on an Linux host system like Ubuntu 24.04. It supports armhf, arm64, amd64, and armv6 targets.

---

## 🔧 Requirements (on Host)

- Ubuntu 24.04 (host)
- Docker installed (`sudo apt install docker.io`)
- QEMU registered for multi-arch builds:
  ```bash
  docker run --rm --privileged multiarch/qemu-user-static --reset -p yes
  ```

---

## 📁 Directory Overview

```
volumio-bluetooth-core/
├── build/
│   └── bluez/                      # BlueZ source and packaging
│       ├── source/                 # BlueZ 5.72 source
│       ├── debian/                 # Debian packaging metadata
│       └── patches/                # (optional) patches for customizations
├── docker/
│   ├── Dockerfile.bluez.amd64      # Dockerfile for amd64 (x86_64) architecture
│   ├── Dockerfile.bluez.arm64      # Dockerfile for ARM64 architecture
│   ├── Dockerfile.bluez.armhf      # Dockerfile for ARMv7 (armhf) architecture
│   ├── Dockerfile.bluez.armv6      # Dockerfile for ARMv6 architecture
│   └── run-docker-bluez.sh         # Script to run Docker builds for different architectures
├── docs/
│   ├── build-bluez.md              # Build guide for BlueZ
│   └── integration.md              # Integration guide for Volumio and BlueZ
├── out/
│   ├── amd64/                      # Output directory for amd64 .deb packages
│   ├── arm64/                      # Output directory for ARM64 .deb packages
│   ├── armhf/                      # Output directory for ARMv7 (armhf) .deb packages
│   └── armv6/                      # Output directory for ARMv6 .deb packages
├── package-sources/
│   ├── bluez_5.72-1.debian.tar.xz  # Debian packaging files for BlueZ
│   └── bluez_5.72.orig.tar.xz      # Original source tarball for BlueZ 5.72
├── README.md                       # Project README file
└── scripts/
    └── extract-bluez-source.sh     # Script to prepare BlueZ source
```

---

## 🧭 What Happens Where

| Context       | Purpose                         |
|---------------|---------------------------------|
| Host (Ubuntu) | Edits files, runs Docker builds |
| Docker        | Runs dpkg-buildpackage          |
| Volumio OS    | Installs final .deb packages    |

---

## 🧱 Step-by-Step: Build BlueZ

### 1. Prepare Source & Packaging

Start by running the `extract-bluez-source.sh` script, which will prepare the BlueZ source and packaging files:

```bash
# Execute the script to extract BlueZ source and prepare packaging
./scripts/extract-bluez-source.sh
```

This will:
- Extract BlueZ 5.72 source.
- Extract it into `build/bluez/source/`.
- Copy the necessary Debian packaging files into the `build/bluez/debian/` directory.

After running the script, continue with the following steps.

### 2. Edit debian/changelog

Navigate to the `debian/` directory and update the changelog to reflect the custom Volumio build:

```bash
# Edit the changelog to add a custom Volumio version
nano build/bluez/debian/changelog
```

Add this entry at the top:
```
bluez (5.72-1volumio1) bookworm; urgency=medium

  * Volumio-customized BlueZ build

 -- Your Name <you@example.com>  Tue, 26 Mar 2024 12:00:00 +0000
```

### 3. Build for an Arch

Once the source and changelog are prepared, you can proceed to build BlueZ for your desired architecture.

To build for a specific architecture, run the appropriate command with `--verbose` flags for more detailed output:

```bash
# Standard build for ARMv7 (armhf)
./docker/run-docker-bluez.sh bluez armhf --verbose

# Volumio-style .deb suffix
./docker/run-docker-bluez.sh bluez armhf volumio --verbose

# Standard build for ARMv8 (arm64)
./docker/run-docker-bluez.sh bluez arm64 --verbose

# Standard build for x86_64 (amd64)
./docker/run-docker-bluez.sh bluez amd64 --verbose

# Standard build for ARMv6
./docker/run-docker-bluez.sh bluez armv6 --verbose
```

Output will be generated in the `out/` directory; example output:

```
out/armhf/bluez_5.72-1volumio1_arm.deb
out/armhf/libbluetooth3_5.72-1volumio1_arm.deb
out/arm64/bluez_5.72-1volumio1_arm64.deb
out/amd64/bluez_5.72-1volumio1_x64.deb
out/armv6/bluez_5.72-1volumio1_armv6.deb
```

---

## 📦 On Volumio OS (Target)

To install the generated `.deb` packages on Volumio OS:

```bash
dpkg -i out/armhf/bluez_5.72-1volumio1_arm.deb
dpkg -i out/armhf/libbluetooth3_5.72-1volumio1_arm.deb
apt-mark hold bluez libbluetooth3
```

For other architectures, replace `armhf` with `arm64`, `amd64`, or `armv6` as appropriate:

```bash
dpkg -i out/arm64/bluez_5.72-1volumio1_arm64.deb
dpkg -i out/amd64/bluez_5.72-1volumio1_x64.deb
dpkg -i out/armv6/bluez_5.72-1volumio1_armv6.deb
```

---

## ✅ Repeat for Other Arches

You can repeat the build process for other architectures like `arm64`, `amd64`, and `armv6`:

```bash
./docker/run-docker-bluez.sh bluez arm64 volumio --verbose
./docker/run-docker-bluez.sh bluez amd64 volumio --verbose
./docker/run-docker-bluez.sh bluez armv6 volumio --verbose
```

---

### Additional Notes:
- If you're building for multiple architectures, ensure the appropriate Dockerfile (`Dockerfile.bluez.arm64`, `Dockerfile.bluez.amd64`, etc.) is used.
- The `extract-bluez-source.sh` script should be rerun if you want to update the source or packaging files.
- Use the `--verbose` flag in the `run-docker-bluez.sh` commands to get detailed logs during the build process.
