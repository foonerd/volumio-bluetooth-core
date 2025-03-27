# Build Guide: BlueZ-ALSA Utils for Volumio (Multi-Arch)

This guide outlines how to build **bluez-alsa-utils 4.3.1** for Volumio using Docker on an Ubuntu 24.04 host system. It supports armhf, arm64, amd64, and armv6 targets.

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
│   └── bluez-alsa-utils/                   # bluez-alsa-utils source and packaging
│       ├── source/                         # bluez-alsa-utils source code
│       ├── debian/                         # Debian packaging metadata
│       └── patches/                        # Optional patches for customizations
├── docker/
│   ├── Dockerfile.alsa.armhf               # Dockerfile for ARMv7 (armhf)
│   ├── Dockerfile.alsa.arm64               # Dockerfile for ARMv8 (arm64)
│   ├── Dockerfile.alsa.amd64               # Dockerfile for x86_64 (amd64)
│   ├── Dockerfile.alsa.armv6               # Dockerfile for ARMv6
│   └── run-docker.sh                       # Script to run Docker builds for various architectures
├── out/
│   └── <arch>/                             # .deb outputs for each architecture
├── package-sources/
│   ├── bluez-alsa_4.3.1-1.debian.tar.xz    # Debian packaging files for bluez-alsa-utils
│   └── bluez-alsa_4.3.1.orig.tar.xz        # Original source tarball for bluez-alsa-utils
├── README.md                               # Project README file
└── scripts/
    └── extract-bluez-alsa-utils-source.sh  # Script to prepare bluez-alsa-utils source
```

---

## 🧭 What Happens Where

| Context       | Purpose                         |
|---------------|---------------------------------|
| Host (Ubuntu) | Edits files, runs Docker builds |
| Docker        | Runs dpkg-buildpackage          |
| Volumio OS    | Installs final .deb packages    |

---

## 🧱 Step-by-Step: Build BlueZ-ALSA Utils

### 1. Prepare Source & Packaging

Start by running the `extract-bluez-alsa-utils-source.sh` script to fetch the necessary source files and Debian packaging metadata for **bluez-alsa-utils**:

```bash
# Execute the script to extract bluez-alsa-utils source and prepare packaging
./scripts/extract-bluez-alsa-utils-source.sh
```

This script will:
- Extract **bluez-alsa 4.3.1** source.
- Set up the required packaging metadata for building the **bluez-alsa-utils** package.

After running the script, continue with the following steps.

### 2. Edit debian/changelog

Navigate to the `debian/` directory and update the changelog to reflect the custom Volumio build:

```bash
# Edit the changelog to add a custom Volumio version
nano build/bluez-alsa-utils/debian/changelog
```

Add this entry at the top:
```
bluez-alsa-utils (4.3.1-1volumio1) bookworm; urgency=medium

  * Volumio-customized bluez-alsa-utils build

 -- Your Name <you@example.com>  Tue, 26 Mar 2024 12:00:00 +0000
```

### 3. Build for an Arch

Once the source and changelog are prepared, you can proceed to build **bluez-alsa-utils** for your desired architecture. Run the appropriate command with `--verbose` flags for detailed output:

```bash
# Build for ARMv7 (armhf)
./docker/run-docker-alsa-utils.sh bluez-alsa-utils armhf --verbose

# Build for ARMv8 (arm64)
./docker/run-docker-alsa-utils.sh bluez-alsa-utils arm64 --verbose

# Build for x86_64 (amd64)
./docker/run-docker-alsa-utils.sh bluez-alsa-utils amd64 --verbose

# Build for ARMv6
./docker/run-docker-alsa-utils.sh bluez-alsa-utils armv6 --verbose
```

Output will be generated in the `out/` directory:

```
out/armhf/bluez-alsa-utils_4.3.1-1volumio1_arm.deb
out/arm64/bluez-alsa-utils_4.3.1-1volumio1_arm64.deb
out/amd64/bluez-alsa-utils_4.3.1-1volumio1_x64.deb
out/armv6/bluez-alsa-utils_4.3.1-1volumio1_armv6.deb
```

---

## 📦 On Volumio OS (Target)

To install the generated `.deb` packages on Volumio OS:

```bash
# Example for ARMv7 (armhf)
dpkg -i out/armhf/bluez-alsa-utils_4.3.1-1volumio1_arm.deb
apt-mark hold bluez-alsa-utils bluez-alsa-utils-common
```

For other architectures, replace `armhf` with `arm64`, `amd64`, or `armv6` as appropriate:

```bash
dpkg -i out/arm64/bluez-alsa-utils_4.3.1-1volumio1_arm64.deb
dpkg -i out/amd64/bluez-alsa-utils_4.3.1-1volumio1_x64.deb
dpkg -i out/armv6/bluez-alsa-utils_4.3.1-1volumio1_armv6.deb
```

---

## ✅ Repeat for Other Arches

You can repeat the build process for other architectures like `arm64`, `amd64`, and `armv6`:

```bash
./docker/run-docker-alsa-utils.sh bluez-alsa-utils arm64 volumio --verbose
./docker/run-docker-alsa-utils.sh bluez-alsa-utils amd64 volumio --verbose
./docker/run-docker-alsa-utils.sh bluez-alsa-utils armv6 volumio --verbose
```

---

### Additional Notes:
- If you're building for multiple architectures, ensure the appropriate Dockerfile (`Dockerfile.alsa.arm64`, `Dockerfile.alsa.amd64`, etc.) is used.
- The `extract-bluez-alsa-utils-source.sh` script should be rerun if you want to update the source or packaging files.
- Use the `--verbose` flag in the `run-docker-alsa-utils.sh` commands to get detailed logs during the build process.
