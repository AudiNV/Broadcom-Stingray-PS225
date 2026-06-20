# EDK2 Runtime Scripts Analysis

## Overview

Recovered EDK2 runtime scripts provide valuable insight into the operational architecture of the Broadcom Stingray PS225 platform.

Rather than functioning as a simple NIC firmware environment, the platform appears to operate as a complete ARM64 system utilizing:

- EDK2 UEFI firmware
- EFI variable-based configuration
- eMMC storage
- Recovery and rollback mechanisms
- Multi-slot image management
- Network-based firmware updates

These scripts appear to form the primary management framework used by the onboard ARM subsystem.

---

# EFI Variable Framework

The platform stores configuration and boot state information using EFI variables.

Two EFI namespaces are supported:

```text
GlobalVariableGuid
8be4df61-93ca-11d2-aa0d-00e098032b8c

BroadcomVariableGuid
c075edd3-681a-4869-8a54-606751c46f4e
```

Runtime scripts automatically detect which namespace is active and use the appropriate storage location.

---

## Observed EFI Variables

The following variables have been observed within the runtime management framework:

```text
bcm_boot_count
bcm_boot_recovery

bcm_dtb_slot
bcm_dtb_slots_count

bcm_kernel_slot
bcm_kernel_slots_count

bcm_rootfs_slot
bcm_rootfs_slots_count

bcm_inband_ipaddr
bcm_inband_netmask
bcm_inband_gatewayip
```

These variables collectively control:

- Boot state
- Recovery state
- Device Tree selection
- Kernel selection
- Root filesystem selection
- Network configuration

---

# Recovery Architecture

Recovery mode is controlled entirely through EFI variables.

To force recovery mode:

```text
bcm_boot_recovery = 1
```

To return to normal operation:

```text
bcm_boot_recovery = 0
bcm_boot_count = 0
```

No hardware jumper, DIP switch, or physical recovery button has yet been identified as being required.

Recovery appears to be implemented as a firmware-controlled state machine.

---

# Boot Slot Architecture

The platform supports independent slot management for:

```text
Device Tree (DTB)
Kernel
Root Filesystem
```

Observed helper functions include:

```text
get_next_slot("dtb")
get_next_slot("kernel")
get_next_slot("rootfs")
```

This strongly suggests an A/B style update system designed to allow:

- Safe updates
- Rollback capability
- Recovery from failed upgrades

---

# Firmware Update Framework

The update environment supports downloading firmware components over the network.

Observed protocols:

```text
TFTP
HTTP
HTTPS
```

Supported update targets include:

```text
dtb
fip
kernel
rootfs
```

Integrity validation is performed before installation.

Observed checks include:

```text
MD5SUM validation
Image integrity verification
Partition validation
```

The update process contains extensive error handling for:

- Invalid images
- Incorrect partitions
- Active rootfs protection
- Recovery image protection
- Failed downloads
- Failed writes

---

# Storage Architecture

## eMMC

Observed storage devices:

```text
/dev/mmcblk0boot0
/dev/mmcblk0p1
/dev/mmcblk0pX
```

---

## EFI System Partition (ESP)

Observed partition:

```text
/dev/mmcblk0p1
```

Contents appear to include:

```text
EFI boot files
Kernel images
DTB images
run_once.nsh
```

---

## Boot Partition

Observed device:

```text
/dev/mmcblk0boot0
```

Contains:

```text
FIP (Firmware Image Package)
```

Observed write offset:

```text
0x20000
```

---

# UEFI Shell Integration

The runtime framework automatically generates UEFI shell scripts.

Observed file:

```text
run_once.nsh
```

Generated commands include:

```text
rtvar bcm_boot_recovery
rtvar bcm_kernel_slot
rtvar bcm_rootfs_slot
```

This confirms active use of the UEFI shell environment during system operation and updates.

---

# Nitro References

Several scripts reference the term:

```text
Nitro
```

Examples include:

```text
dump_nitro_freq.sh
Nitro image updates
Nitro interface detection
```

Current significance remains under investigation.

Possible interpretations include:

- ARM subsystem codename
- Dataplane subsystem
- Firmware package identifier
- Internal Broadcom project name

Additional evidence is required before drawing conclusions.

---

# Clock Monitoring

Engineering-oriented diagnostic scripts provide direct hardware register access.

Observed functionality:

```text
PLL inspection
Frequency calculation
Clock monitoring
```

Example outputs include:

```text
VCO Frequency
Nitro Frequency
```

This suggests Broadcom intentionally exposed low-level hardware diagnostics within the Linux userspace environment.

---

# Platform Management Functions

Recovered scripts provide functionality for:

```text
Recovery Mode Control
Normal Boot Control
EFI Variable Management
Slot Selection
Firmware Updates
Integrity Verification
Network Configuration
Clock Monitoring
System Diagnostics
```

Collectively these scripts form a complete platform management layer.

---

# Preliminary Boot Flow

Current evidence suggests the following boot architecture:

```text
ROM
 ↓
Trusted Firmware (FIP)
 ↓
EDK2 UEFI
 ↓
EFI Variables
 ↓
Recovery / Slot Selection Logic
 ↓
Linux ARM64
 ↓
Broadcom Runtime Management Scripts
```

Further investigation is required to fully document the Trusted Firmware and FIP components.

---

# Significance

The recovered runtime scripts demonstrate that the Broadcom Stingray PS225 platform is substantially more sophisticated than a conventional SmartNIC.

The platform contains:

- Persistent EFI variable storage
- UEFI shell support
- eMMC-based operating system storage
- Multi-slot image management
- Recovery infrastructure
- Network-based update mechanisms
- Low-level hardware diagnostics

These findings support the conclusion that the PS225 operates as a complete ARM64 embedded computing platform integrated into a PCIe SmartNIC form factor.

---

# Current Confidence Level

## Confirmed

- EDK2 UEFI firmware environment
- EFI variable-based configuration
- Recovery mode support
- Multi-slot image management
- eMMC storage architecture
- Network-based firmware update capability
- UEFI shell integration

## Under Investigation

- FIP contents
- Nitro subsystem role
- Secure Boot implementation details
- Image signing requirements
- Firmware rollback logic
- Trusted Firmware components

---

Last Updated: June 2026
Source: Recovered EDK2 runtime scripts from Broadcom Stingray PS225 software package.
