# Dell Latitude E6400 — macOS High Sierra (10.13.6) Hackintosh Guide (PoC)

![Status](https://img.shields.io/badge/Status-Proof%20of%20Concept%20%2F%20Untested-yellow?style=flat-square)
![macOS](https://img.shields.io/badge/Target%20OS-macOS%20High%20Sierra%2010.13.6-blue?style=flat-square&logo=apple)
![Bootloader](https://img.shields.io/badge/Bootloader-OpenCore%20%7C%20Clover-lightgrey?style=flat-square)
![Firmware](https://img.shields.io/badge/Firmware-Legacy%20BIOS%20(DuetPkg)-orange?style=flat-square)

> **⚠️ Status Notice:** This repository is currently a **Proof-of-Concept (PoC) and theoretical architectural analysis**. No pre-built `EFI` folder is included, and these configurations have not yet been live-tested on physical hardware. Contributions, test reports, and pull requests from E6400 owners are welcome!

A comprehensive technical analysis, hardware compatibility breakdown, and theoretical implementation roadmap for deploying **macOS High Sierra (10.13.6)** on the **Dell Latitude E6400** (NVIDIA Quadro NVS 160M variant).

> **📖 Full Technical Paper:** Read [`GUIDE.md`](./GUIDE.md) for the complete architectural deep-dive covering ACPI/DSDT patching, DuetPkg MBR-to-UEFI emulation, AGPM thermal tuning, and troubleshooting.

---

## Hardware Compatibility Matrix (Research Baseline)

Based on community documentation (OSXLatitude, InsanelyMac, and Dortania), the **NVIDIA Quadro NVS 160M** discrete GPU and an **SSE4.1-capable Core 2 Duo** processor are mandatory for a usable macOS High Sierra installation. Models equipped solely with the integrated Intel GMA 4500MHD lack Quartz Extreme / Core Image (QE/CI) acceleration.

| Component | Specification | Expected Status | Theoretical Implementation Strategy |
| :--- | :--- | :---: | :--- |
| **Processor** | Intel Core 2 Duo P8700 (2.53 GHz, Penryn) | Supported (Legacy) | `ssdtPRGen.sh` / `CPUFriend.kext` (OpenCore) or native P/C-States (Clover) |
| **Chipset** | Intel PM45 Express | Supported | LPC bus spoofing via DSDT / `AppleLPC` |
| **Memory** | 8 GB DDR2 800 MHz (2 x 4 GB) | Supported | BIOS maximum supported capacity |
| **Discrete GPU** | NVIDIA Quadro NVS 160M (256 MB, Tesla) | Supported | NVIDIA Web Drivers (`387.10.10.10.40.136`) + AGPM `control-id` 18 |
| **Integrated GPU** | Intel GMA 4500MHD | Unsupported | No QE/CI drivers exist in macOS; VESA mode only |
| **Storage** | SATA III SSD | Supported | BIOS set to `AHCI` + `sudo trimforce enable` |
| **Audio Codec** | IDT 92HD71B7 | Supported | `AppleALC.kext` (Layout `11`, `12`, or `13` for E-Port docks) + DSDT HPET IRQ fix |
| **Ethernet** | Intel 82567LM Gigabit | Supported | `IntelMausi.kext` |
| **Wi-Fi (Upgrade)** | Intel Wi-Fi 6 AX200 *(via M.2-to-mPCIe adapter)* | Supported | `itlwm.kext` + `HeliPort.app` (or native AirPort via legacy Broadcom DW1510) |
| **Bluetooth** | Internal Broadcom / Intel module | Unsupported | Internal half-mini PCIe slot lacks USB data lanes |
| **External Video** | VGA & E-Port Dock Digital Out | Supported | Built-in DisplayPort is non-functional under macOS |
| **Trackpad / Keyboard** | ALPS PS/2 Trackpad & Keyboard | Supported | ALPS-specific fork of `VoodooPS2Controller.kext` (nested `VoodooInput.kext`) |

---

## Target BIOS Configuration (Press `F2` at Boot)

Before attempting an installation, the Dell BIOS should be configured as follows to ensure proper ACPI and SATA hand-off:

- **System Configuration -> SATA Operation:** `AHCI` *(Mandatory; IRRT and ATA will cause kernel panics / root device timeouts)*
- **Performance -> Multi-Core Support:** `Enabled`
- **Performance -> SpeedStep:** `Enabled` *(Required for CPU frequency scaling)*
- **Performance -> Execute Disable Bit:** `Enabled` *(Prevents early kernel boot loops)*
- **Security -> Secure Boot / TPM:** `Disabled`
- **Virtualization -> VT-x:** `Enabled`
- **POST Behavior -> Fastboot:** `Thorough` *(Ensures full USB and SATA node enumeration)*

---

## Repository Structure

```text
Dell-Latitude-E6400-Hackintosh/
├── README.md                                      # Project overview, PoC status, and hardware matrix
├── GUIDE.md                                       # Full architectural and implementation manual
├── Tools-and-Scripts/
│   └── POST-INSTALL-NOTES.md                      # Reference AGPM plist, alc-verb, & NVRAM notes
└── docs/
    └── Hackintosh Dell Latitude E6400 Guide.pdf   # Original PDF compilation
```

---

## Contributing & Testing

If you own a **Dell Latitude E6400 (NVS 160M)** and test any part of this guide—or have a working, sanitized OpenCore/Clover `EFI` folder you'd like to contribute—feel free to open an **Issue** or **Pull Request** to help turn this PoC into a verified community resource.

## Disclaimer

This repository contains theoretical documentation, configuration notes, and references to open-source bootloader tools only. **No proprietary Apple binaries, macOS installers, or copyrighted firmware images are hosted here.**