## Mac-to-Linux Mint OS Migration & Optimization

A technical walkthrough, system configuration guide, and hardware optimization record for converting legacy Apple hardware into a high-performance development workstation running **Linux Mint XFCE**.

---

## Table of Contents
- [Project Overview](#project-overview)
- [Target Hardware Specifications](#target-hardware-specifications)
- [Key Engineering Accomplishments](#key-engineering-accomplishments)
- [Technical Procedures & Hardware Configurations](#technical-procedures--hardware-configurations)
  - [1. Storage Audit & System Reclamation](#1-storage-audit--system-reclamation)
  - [2. Kernel Bootloader Parameters (GRUB)](#2-kernel-bootloader-parameters-grub)
  - [3. Display Blanking & X11 Driver Workarounds](#3-display-blanking--x11-driver-workarounds)
  - [4. Custom systemd Power & Runtime Management](#4-custom-systemd-power--runtime-management)
  - [5. Apple Hardware Driver Integration (Audio & Wi-Fi)](#5-apple-hardware-driver-integration-audio--wi-fi)
- [System Verification & Diagnostic Reports](#system-verification--diagnostic-reports)
- [Repository Structure](#repository-structure)
- [License](#license)

---

## Project Overview

This project documents the complete software conversion and system-level optimization of a 2017 13-inch MacBook Pro running Linux Mint XFCE.

Deploying modern Linux distributions to mid-2010s Apple hardware presents specific engineering challenges: NVMe storage power-state freezes, ACPI resource locking, display backlight blanking bugs under X11/lightdm, sleep/suspend loop crashes, and unconfigured proprietary audio/wireless chipsets. This repository details the exact shell commands, kernel flags, driver configurations, and `systemd` overrides implemented to resolve these hardware edge cases and establish a highly responsive, stable Linux workstation.

---

## Target Hardware Specifications

- **Device:** Apple MacBook Pro (13-inch, 2017 / `MacBookPro14,1`)
- **Host OS:** Linux Mint 22.3 Zena (XFCE 4.18.1)
- **Kernel:** Linux 6.8.0 LTS
- **Core Objectives:** Power latency tuning, ACPI resource management, non-suspend display power management (DPMS), proprietary driver integration, and system storage reclamation.

---

## Key Engineering Accomplishments

- **Reclaimed 13 GB of Storage Space:** Conducted low-level disk usage audits using `journalctl`, `apt`, and `ncdu` to purge orphaned package dependencies, vacuum system logs, and optimize system caches.
- **NVMe & ACPI Kernel Tuning:** Applied custom GRUB kernel parameters (`nvme_core.default_ps_max_latency_us=5500` and `acpi_enforce_resources=lax`) to prevent SSD latency crashes and resolve resource conflicts on Apple motherboard hardware.
- **X11 Display Blanking Workaround:** Overcame display power management bugs under XFCE/lightdm by implementing direct kernel-level console blanking commands via `setterm`, bypassing problematic X11 display timeouts.
- **Custom `systemd` Power Policies:** Configured system-wide `systemd` logind override targets to strictly enforce a non-suspend policy while maintaining automated screen blanking and power management.
- **Hardware Chipset Driver Resolution:** Resolved driver incompatibilities for the Cirrus Logic audio chip and Broadcom wireless modules (`brcmfmac`) to restore native sound output and stable network performance.

---

## Technical Procedures & Hardware Configurations

### 1. Storage Audit & System Reclamation
Executed targeted root-level system audits to isolate non-essential storage bloat and optimize package dependencies:
```bash
# Purge orphaned package dependencies and clear APT package cache
sudo apt-get autoremove --purge -y
sudo apt-get clean

# Vacuum systemd journal logs and enforce a strict 100MB retention cap
sudo journalctl --vacuum-size=100M

```

### 2. Kernel Bootloader Parameters (GRUB)

Configured GRUB boot parameters in `/etc/default/grub` to fix NVMe SSD power latency drops and relax ACPI resource enforcement for Apple hardware:

```text
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash nvme_core.default_ps_max_latency_us=5500 acpi_enforce_resources=lax"

```

*Applied to bootloader via `sudo update-grub`.*

### 3. Display Blanking & X11 Driver Workarounds

To bypass X11/XFCE display sleep bugs on the MacBook screen backlight, low-level virtual terminal blanking was configured to handle display power-down states directly at the console level:

```bash
# Force console screen blanking and power-down via setterm (10-minute timeout)
setterm -blank 10 -powerdown 10

```

### 4. Custom systemd Power & Runtime Management

To prevent system suspend/sleep crash loops on Apple hardware while allowing display turn-off, `systemd` power targets were configured in `/etc/systemd/logind.conf`:

```ini
[Login]
HandleLidSwitch=ignore
HandleLidSwitchExternalPower=ignore
HandleLidSwitchDocked=ignore
IdleAction=ignore

```

### 5. Apple Hardware Driver Integration (Audio & Wi-Fi)

* **Audio Chipset Patching:** Implemented driver configuration patches for the Cirrus Logic audio chip to restore native speaker output and headphone jack switching.
* **Broadcom Wireless Synchronization:** Configured Broadcom Wi-Fi module (`brcmfmac`) power management settings to eliminate connection dropping and maintain network stability across reboots.

---

## System Verification & Diagnostic Reports

System state and hardware verification reports can be generated directly from the terminal using built-in system diagnostics tools:

```bash
# Export system configuration summary
inxi -Fz > HARDWARE-REPORT.md

```

The generated report confirms active kernel releases, active driver modules, memory usage, and storage allocation directly from live hardware.

---

## Repository Structure

```
├── HARDWARE-REPORT.md        # System diagnostic dump generated via inxi
├── LICENSE                   # MIT License
└── README.md                 # Technical walkthrough, commands, and configurations

```

---

## License

This repository is available open-source under the [MIT License](https://www.google.com/search?q=LICENSE&utm_source=gemini).
