# Broadcom Stingray PS225 Lab

Community hardware/software notes for the Broadcom Stingray PS225 SmartNIC / DPU platform.

Repository target:

```text
https://github.com/AudiNV/Broadcom-Stingray-PS225
```

## Current Hardware

- 2x PS225-H16 cards
- 2x PS225-H08 cards
- 1x official Broadcom USB-to-3.5mm TRRS serial console cable
- Broadcom PS225 / BCM5880X documentation archive

## Inventory Summary

| Card | Assembly Number | Model | Revision | Expected Memory | Status |
|---|---|---|---|---:|---|
| Card 01 | BCM958802A8046C | PS225-H16 | Rev 17 | 16GB | Untested |
| Card 02 | BCM958802A8046C | PS225-H16 | Rev 15 | 16GB | Untested |
| Card 03 | BCM958802A8028E | PS225-H08 | Rev E | 8GB ECC estimated | Untested |
| Card 04 | BCM958802A8028C | PS225-H08 | Rev C | 8GB | Untested |

See [`hardware/inventory.md`](hardware/inventory.md) for detailed card-by-card information.

## Key Platform Features

- Broadcom Stingray BCM5880X / BCM58802H SoC
- 8x ARM Cortex-A72 cores at 3.0GHz
- PCIe 3.0 x8 host interface
- Dual 25GbE SFP28 ports
- SR-IOV support
- RoCE v1/v2 support
- TruFlow flow acceleration
- Line-rate crypto engine
- RAID5/6 / erasure acceleration engine
- Onboard eMMC storage
- Onboard SPI flash
- 3.5mm TRRS serial console

## Repo Layout

```text
docs/                  Broadcom and component reference documents
hardware/              Card photos, labels, inventory, cable notes
bootlogs/              Captured serial boot logs
scripts/               Host-side collection scripts
firmware/              Firmware notes and dumps; do not commit proprietary blobs blindly
software/              Driver / userspace notes
reverse-engineering/   Boot, flash, partition, and recovery findings
research/              Analysis notes
```

## Current Focus

1. Preserve hardware identification details.
2. Capture serial boot logs from all cards.
3. Capture host PCIe enumeration.
4. Dump original eMMC/SPI contents from working cards before flashing anything.
5. Compare H08 vs H16 images and NVRAM configuration.

# Broadcom Stingray PS225 Research Project

## Overview

This repository documents the reverse engineering, analysis, firmware investigation, and bring-up efforts surrounding the Broadcom Stingray PS225 SmartNIC platform.

The goal of this project is to identify hardware variants, document firmware architecture, understand platform capabilities, and ultimately bring previously undocumented PS225 cards into operational use under Linux.

---

## What Is The PS225?

The PS225 is not a traditional Ethernet adapter.

Evidence collected from Linux drivers, Broadcom documentation, firmware interfaces, and community research indicates the PS225 is a member of the Broadcom Stingray family of SmartNICs / DPUs.

At its core is the:

- Broadcom BCM58802 Stingray SoC
- ARM Cortex-A72 subsystem
- Onboard DDR4 memory
- Onboard eMMC storage
- 25GbE network interfaces
- RoCE / RDMA acceleration
- SR-IOV virtualization support
- Secure Boot infrastructure

---

## Hardware Identified

### ASIC

Vendor ID:

```
14e4:d802
```

Linux Identification:

```
Broadcom BCM58802 Stingray 50Gb Ethernet SoC
```

Driver:

```
bnxt_en
```

---

## Known Memory Configurations

Evidence currently suggests the existence of at least:

| DDR4 | eMMC |
|--------|--------|
| 8GB | 16GB |
| 16GB | 64GB |

Additional variants may exist.

---

## Broadcom Stingray Architecture

Based on Broadcom public documentation:

- Up to 8x ARM Cortex-A72 cores
- PCIe Gen3
- Integrated Ethernet controller
- RoCE acceleration
- NVMe-over-Fabric support
- Hardware packet processing
- 16nm FinFET process

---

# Firmware Architecture

Unlike conventional NICs, the PS225 appears to contain multiple independent firmware domains.

## CHIMP

Responsible for:

- HWRM processing
- Management
- Configuration
- Firmware services

## KONG

Responsible for:

- Dataplane processing
- Flow management
- RoCE services
- CFA / TruFlow operations

## Additional Firmware Components

Discovered through Broadcom NVM definitions:

- APE Firmware
- KONG Firmware
- BONO Firmware
- TANG Firmware

Each component supports independent firmware and patch images.

---

# HWRM Capability Discovery

The Broadcom HWRM interface exposes the following capabilities.

## Security

- Secure Firmware Updates
- Secure Boot Capable
- Secure SoC Capable
- Debug Token Support

## Virtualization

- Trusted VF Support
- VirtIO vSwitch Offload

## Dataplane

- KONG Mailbox Channel
- Flow Aging
- Advanced Flow Counters
- CFA Advanced Flow Management
- CFA TruFlow

---

# NVM Layout

Broadcom stores firmware and configuration inside a directory-based NVM structure.

Known object types include:

| Type | Description |
|--------|--------|
| BOOTCODE | Bootloader |
| VPD | Vital Product Data |
| APE_FW | APE Firmware |
| KONG_FW | KONG Firmware |
| BONO_FW | BONO Firmware |
| TANG_FW | TANG Firmware |
| SHARED_CFG | Shared Configuration |
| PORT_CFG | Port Configuration |
| FUNC_CFG | Function Configuration |
| MGMT_CFG | Management Configuration |
| MGMT_DATA | Management Data |
| MGMT_EVENT_LOG | Event Log |
| MGMT_AUDIT_LOG | Audit Log |

---

# RoCE / RDMA

The Linux driver exposes structures associated with:

- Queue Pairs (QP)
- Completion Queues (CQ)
- Shared Receive Queues (SRQ)
- Memory Regions (MR)

This confirms the presence of hardware RDMA support.

---

# Secure Platform Features

The BCM58802 firmware advertises:

- Secure Boot Capable
- Secure SoC Capable
- Debug Token Support

Additional manufacturing and provisioning commands exist for:

- Certificate import/export
- OTP configuration
- Secure firmware updates

This strongly suggests the platform implements a hardware root-of-trust architecture.

---

# Debug Infrastructure

Broadcom exposes a large engineering command set not normally used by standard operating systems.

Examples include:

- Direct register access
- Indirect register access
- Firmware CLI
- I²C access
- NVM erase operations
- Crash dump generation
- Trace facilities
- Debug token management

These commands appear within a dedicated debug opcode range:

```
0xFF0E - 0xFF30
```

---

# Crash Dump Framework

The Stingray platform contains a sophisticated diagnostic framework capable of collecting:

- Firmware dumps
- Driver dumps
- SoC DDR dumps
- Host DDR dumps

Known dumpable components include:

- KONG
- HWRM
- VNIC
- RDMA Queues
- Statistics Contexts

---

# RouterOS Discovery

Community reports have demonstrated:

- RouterOS CHR running directly on a PS225-H16
- Proxmox remaining operational on the host
- Network functions exposed to virtual machines

This confirms that at least some Stingray variants are capable of executing complete operating systems on the ARM subsystem.

---

# Current Status

## Confirmed

- BCM58802 Stingray SoC
- CHIMP Management Processor
- KONG Dataplane Processor
- RoCE Support
- SR-IOV Support
- Secure Firmware Updates
- Secure Boot Capability
- DDR4 + eMMC Architecture

## Under Investigation

- Firmware image formats
- Boot process
- Secure provisioning flow
- Debug token infrastructure
- Operating system deployment methods
- NVM image extraction and analysis

---

# Project Goals

1. Inventory all available PS225 hardware.
2. Dump and compare firmware images.
3. Document firmware layout.
4. Bring cards online under Linux.
5. Investigate ARM-side operating system support.
6. Document recovery procedures.
7. Preserve knowledge for the broader homelab and research community.

---

# Disclaimer

This repository contains research and documentation generated from publicly available sources, Linux driver analysis, community findings, and direct hardware inspection.

No proprietary Broadcom source code is included.

All trademarks belong to their respective owners.

---

# Contributors

Contributions, documentation, photographs, firmware analysis, and test results are welcome.

If you have a PS225, BCM58802, BCM58804, or other Stingray-based platform, please open an issue and share your findings.

## Caution

Do not flash or overwrite any card until the original boot logs, PCI IDs, eMMC layout, SPI/NVRAM contents, and firmware versions have been captured.
