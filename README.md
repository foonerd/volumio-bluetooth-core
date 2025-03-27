Here's a draft for the **main `README.md`** that focuses on the Bluetooth issue within Volumio, with a clear explanation of the project, its goals, and the impact of Bluetooth on the overall system performance, including volume behavior.

---

# Volumio Bluetooth Core

The **Volumio Bluetooth Core** project is designed to provide a streamlined and high-quality Bluetooth audio experience for Volumio OS, focused on improving Bluetooth audio handling, volume behavior, and system integration. It tackles longstanding Bluetooth issues, such as metadata transmission, volume synchronization, and ensuring optimal audio quality for audiophile use.

## 🎯 Purpose

Volumio is a high-quality audio streaming solution designed for audiophiles, prioritizing the shortest possible audio path between the music source and sound output. Bluetooth support is a critical feature, allowing users to connect wireless audio devices with Volumio while maintaining high fidelity.

However, Volumio’s current Bluetooth implementation presents several challenges:

- **Volume Behavior**: Controlling the volume across Bluetooth-connected devices has been inconsistent, with some devices not responding properly to volume changes. This project aims to fix bidirectional volume control, ensuring that both Volumio and Bluetooth-connected devices can control volume seamlessly.
  
- **Metadata Handling**: Volumio has faced challenges with Bluetooth metadata, such as track information and playback status, not syncing properly with the UI. This project focuses on improving metadata transmission, ensuring that information like track name, artist, and playback status is consistent across the system.

- **Audio Quality**: Maintaining bit-perfect audio quality via Bluetooth is essential for Volumio's audiophile users. This project ensures that Bluetooth audio streaming uses the A2DP (Advanced Audio Distribution Profile) for high-fidelity playback, avoiding low-quality profiles like HFP (Hands-Free Profile).

## ⚙️ Key Features

- **BlueZ-ALSA Integration**: Replaces the older PulseAudio-based Bluetooth stack with BlueZ-ALSA, reducing audio latency and providing better control over Bluetooth audio output.
  
- **Bidirectional Volume Control**: Ensures that volume changes in Volumio are reflected on Bluetooth devices and vice versa. This feature allows users to control the volume from either the Volumio interface or the connected Bluetooth device.

- **Metadata Synchronization**: Updates the handling of Bluetooth metadata, such as track information, play/pause status, and volume changes. This ensures that the Volumio UI remains up-to-date with what's playing on the Bluetooth-connected device.

- **Bit-Perfect Audio**: Focuses on ensuring that audio output from Volumio remains at the highest quality by using A2DP for Bluetooth connections, while explicitly rejecting low-fidelity profiles like HFP to prevent degradation of the audio stream.

## 🔧 Requirements

- **Volumio OS**: This project targets Volumio OS, which is based on Debian, and requires compatibility with multiple architectures (armhf, arm64, amd64).
- **BlueZ**: The core Bluetooth stack for Linux, specifically version 5.72, for managing Bluetooth connections and streaming.
- **BlueZ-ALSA**: Replaces PulseAudio for Bluetooth audio routing to ensure bit-perfect audio output with low latency.

## 🧭 Architecture Support

This project supports the following architectures:

- ARMv6 (e.g., Raspberry Pi Zero, Raspberry Pi 1)
- ARMv7 (e.g., Raspberry Pi 2, 3, 3B+)
- ARM64 (e.g., Raspberry Pi 4, Raspberry Pi 400)
- x86_64 (e.g., Intel/AMD-based systems)

## 🛠️ Installation

### 1. Prepare the Source

Begin by running the `extract-bluez-source.sh` script to fetch the necessary BlueZ source files and Debian packaging metadata:

```bash
./scripts/extract-bluez-source.sh
```

This script will:
- Download and extract BlueZ 5.72 source.
- Set up the required packaging metadata for building the BlueZ package.

### 2. Build the BlueZ Packages

Once the source is prepared, you can build the BlueZ packages for your desired architecture using the provided Docker setup. Run the following command for each architecture:

```bash
# Build for ARMv7 (armhf)
./docker/run-docker.sh bluez armhf --verbose

# Build for ARMv8 (arm64)
./docker/run-docker.sh bluez arm64 --verbose

# Build for x86_64 (amd64)
./docker/run-docker.sh bluez amd64 --verbose

# Build for ARMv6
./docker/run-docker.sh bluez armv6 --verbose
```

This will create the appropriate `.deb` packages in the `out/` directory for each architecture.

### 3. Install the Packages on Volumio OS

After building the `.deb` packages, install them on your Volumio system:

```bash
# Example for ARMv7 (armhf)
dpkg -i out/armhf/bluez_5.72-1volumio1_arm.deb
dpkg -i out/armhf/libbluetooth3_5.72-1volumio1_arm.deb
apt-mark hold bluez libbluetooth3
```

Repeat the installation for the other architectures as needed.

## 🔄 Future Enhancements

- **Enhanced Party Mode**: Allow multiple Bluetooth devices to stay connected, but only allow playback from the last device that started streaming audio.
- **Automatic Profile Switching**: Implement seamless switching between A2DP and other profiles based on the device capabilities.
- **Advanced Audio Features**: Integrate features like multi-room audio and low-latency streaming for certain use cases.

## 🤝 Contributing

Contributions to improve Bluetooth integration and overall functionality are welcome. Please feel free to fork the repository, make changes, and submit pull requests. Before contributing, ensure your changes are aligned with Volumio's high-quality audio and system integration standards.
