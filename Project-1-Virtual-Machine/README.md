# 🖥️ Project 1 – Create a Virtual Machine
**Windows 11 VM Setup, Configuration & Troubleshooting**

> **Platform:** Oracle VirtualBox 7.2.8 | **Host OS:** Windows 11 Home | **Date:** April 27, 2026  
> **Author:** Aaron Vandhana

---

## 📋 Overview

This project documents the end-to-end process of planning, deploying, and troubleshooting a Windows 11 virtual machine using Oracle VirtualBox on a physical Windows 11 host. It covers hardware compatibility research, software installation, OS deployment via unattended setup, post-installation configuration, and a deliberate troubleshooting scenario with full resolution.

**Skills demonstrated:** Virtualization, OS installation, driver verification, patch management, network testing, snapshot management, and hardware troubleshooting.

---

## 🗂️ Table of Contents

- [Phase 1 – Planning & Requirements Research](#phase-1--planning--requirements-research)
- [Phase 2 – Environment Setup](#phase-2--environment-setup)
- [Phase 3 – OS Installation](#phase-3--os-installation)
- [Phase 4 – Post-Installation Configuration](#phase-4--post-installation-configuration)
- [Phase 5 – Troubleshooting Scenario](#phase-5--troubleshooting-scenario)
- [Troubleshooting Summary](#troubleshooting-summary)
- [Key Takeaways](#key-takeaways)

---

## Phase 1 – Planning & Requirements Research

**Tool Used:** Windows System Information (`msinfo32`)

Before beginning, the host machine's specifications were documented to confirm compatibility with running a virtual machine.

| Component | Specification |
|---|---|
| OS | Microsoft Windows 11 Home (Build 26200) |
| Processor | AMD Ryzen 5 3600X – 6 cores, 12 logical processors |
| RAM | 32.0 GB |
| System Type | x64-based PC |
| BIOS Mode | UEFI |
| Motherboard | ASUS ROG STRIX B550-F GAMING |

**Analysis:** The host machine exceeded all minimum requirements for running a Windows 11 VM. The AMD Ryzen 5 3600X supports AMD-V (hardware virtualization), UEFI was confirmed active, and 32 GB of RAM provided ample headroom. Prior to VM creation, **SVM Mode** (AMD's virtualization toggle) was enabled in UEFI/BIOS under `Advanced > CPU Configuration`.

---

## Phase 2 – Environment Setup

**Tool Used:** Oracle VirtualBox 7.2.8 Installer

VirtualBox 7.2.8 was downloaded directly from [virtualbox.org](https://www.virtualbox.org) to ensure software integrity and launched with administrator privileges.

**Components installed:**
- VirtualBox Application (core)
- VirtualBox USB Support
- VirtualBox Networking (Bridge and Host-Only adapters)
- VirtualBox Python Support

> **Note:** The installer flagged missing Python Core and win32api bindings required for Python scripting support. Since Python scripting was not required for this project, installation proceeded without resolving these dependencies.

**ISO Download:** The Windows 11 x64 ISO (`Win11_25H2_English_x64_v2.iso` – 7.9 GB) was sourced directly from Microsoft's official download page at `microsoft.com/en-us/software-download/windows11`.

- The **"Download Windows Disk Image (ISO) for x64 devices"** option was selected — the correct choice for VirtualBox, as the ISO mounts directly as a virtual disc drive.
- The ARM64 ISO was intentionally avoided, as the host uses an x64 AMD processor and the two are incompatible.

---

## Phase 3 – OS Installation

**Tool Used:** Oracle VirtualBox New VM Wizard, Windows 11 Installer

Upon pointing VirtualBox to the downloaded ISO, it automatically detected:
- OS: Microsoft Windows
- Edition: Windows 11 Home (10.0.26200.8037 / x64 / en-US)

**Unattended Installation was used**, which automated the setup process. Configuration included:

| Setting | Value |
|---|---|
| Username | admin |
| Host Name | Windows11 |
| Guest Additions | Pre-configured via `VBoxGuestAdditions.iso` |

**Virtual Hardware Allocated:**

| Resource | Allocated | Rationale |
|---|---|---|
| Base Memory | 4096 MB (4 GB) | Meets Windows 11 minimum; leaves ample RAM for host |
| CPUs | 4 | Half of available logical processors; balanced performance |
| Disk Size | 100 GB | Dynamic allocation; room for OS, updates, and lab work |
| EFI | Enabled | Required for Windows 11; enables TPM 2.0 support |

> **Why EFI?** Windows 11 requires UEFI firmware and TPM 2.0. Enabling EFI in VirtualBox satisfies this requirement within the virtual environment.

**Result:** Windows 11 installed successfully via unattended setup. The VM booted directly to the Windows 11 desktop. A local `admin` account was created (not a Microsoft account) — preferred for lab environments as it mirrors how domain-joined enterprise machines are typically managed.

---

## Phase 4 – Post-Installation Configuration

### Driver Verification
**Tool:** Device Manager (`devmgmt.msc`)

A full inspection of all device categories confirmed no yellow exclamation marks, red X indicators, or unknown devices. All virtual hardware — network adapters, display adapters, storage controllers, USB controllers, and audio devices — was recognized and loaded successfully.

### Windows Update
**Tool:** `Settings > Windows Update`

5 updates were identified and applied immediately after installation:

| Update | KB Article |
|---|---|
| Microsoft Defender Antivirus Definition Update | KB2267602 |
| .NET Framework Security Update | KB5082417 |
| Cumulative Security Update | KB5083769 |
| Windows Malicious Software Removal Tool | KB890830 |
| Windows Security Platform Update | KB5007651 |

> Applying all available updates immediately after OS installation is standard IT procedure. An unpatched system is vulnerable to known exploits.

### Network Verification
**Tool:** PowerShell (Administrator) — `ping google.com`

| Metric | Result |
|---|---|
| DNS Resolution | google.com → 142.251.41.14 ✅ |
| Packets Sent / Received | 4 / 4 (0% loss) ✅ |
| Average Latency | 14ms ✅ |

The VM successfully communicated through VirtualBox's NAT adapter.

### Baseline Snapshot
Immediately following network verification, a VirtualBox snapshot was taken:

> **Name:** `Clean Baseline – Pre-Troubleshooting`  
> **Description:** "Fully installed, updated, and verified. Safe restore point before new phases."

This snapshot provides a guaranteed recovery point before any intentional changes are made.

---

## Phase 5 – Troubleshooting Scenario

### Problem Introduction
The VM was shut down cleanly via `Start > Power > Shut Down` to preserve filesystem integrity. Base Memory was then intentionally reduced from **4096 MB → 512 MB** in `VirtualBox Settings > System > Motherboard` to simulate a misconfigured VM — a common real-world IT scenario.

### Symptom Observed
Upon booting with 512 MB of RAM, Windows 11 failed to complete the boot process entirely. The Windows Recovery Environment displayed:

```
Error Code: 0xc0000017
"There isn't enough memory available to create a ramdisk device."
```

**Root Cause:** Windows 11 requires a minimum of 4 GB RAM to function. At 512 MB, the system could not initialize the boot ramdisk — a temporary memory-based storage structure required during the Windows boot sequence. This caused a **complete boot failure**, not just degraded performance.

### Resolution Applied
Base Memory was restored from **512 MB → 4096 MB** in `VirtualBox Settings > System > Motherboard` while the VM remained powered off (best practice for modifying virtual hardware).

### Recovery Behavior
Upon reboot with corrected RAM, Windows 11 automatically initiated a self-repair via the Windows Recovery Environment — expected behavior when Windows detects a prior abnormal shutdown. **No manual intervention was required.**

### Resolution Confirmed
**Tool:** Task Manager > Performance Tab

| Metric | Value | Status |
|---|---|---|
| CPU Utilization | 1% | ✅ Normal |
| Memory Usage | 2.5 GB / 4.0 GB (62%) | ✅ Normal |
| Disk Activity | 0% | ✅ No page file swapping |
| Virtual Processors | 4 | ✅ Active |
| System Uptime | 8 min 32 sec | ✅ Stable |

---

## Troubleshooting Summary

| Step | Action | Result |
|---|---|---|
| Problem Introduced | RAM reduced from 4096 MB to 512 MB | VM configured with insufficient memory |
| Symptom Observed | VM failed to boot | Error 0xc0000017 – ramdisk creation failed |
| Root Cause Identified | 512 MB below Windows 11 minimum (4 GB) | OS could not initialize boot sequence |
| Resolution Applied | RAM restored to 4096 MB in VirtualBox Settings | VM booted successfully |
| Resolution Confirmed | Task Manager: 1% CPU, 62% memory | System stable and fully operational |

---

## 💡 Key Takeaways

- **Pre-deployment planning matters.** Verifying host specs and enabling hardware virtualization (SVM) in BIOS before starting prevented compatibility issues downstream.
- **Insufficient RAM doesn't just slow a system — it can prevent it from booting entirely.** Error 0xc0000017 demonstrated this clearly.
- **Snapshots are essential.** The baseline snapshot allowed safe experimentation without risk of permanent damage to the VM.
- **Windows self-healing mechanisms work.** The automatic Recovery Environment repair confirmed the OS integrity was intact and the issue was purely a hardware resource problem.
- **Local accounts vs. Microsoft accounts.** For lab and enterprise simulation environments, local accounts are preferred as they mirror domain-joined machine management without requiring cloud authentication.

---

*Part of the [IT Home Lab Portfolio](../README.md) by Aaron Vandhana*
