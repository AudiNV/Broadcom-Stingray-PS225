# Broadcom Stingray Lab

Community notes and bring-up work for Broadcom Stingray / PS225 SmartNIC DPU cards.

## Focus Hardware

- Broadcom Stingray PS225 SmartNIC
- BCM5880X / BCM58802H SoC
- PS225-H04 / PS225-H08 / PS225-H16 variants
- Dual-port 25GbE SFP28 PCIe 3.0 x8 SmartNICs

## Known PS225 Specs

- 8-core ARMv8 Cortex-A72 processor subsystem at 3.0 GHz
- 4 GB, 8 GB, or 16 GB onboard DDR4 depending on card variant
- Dual SFP28 ports supporting 10GbE/25GbE
- PCIe 3.0 x8 host interface
- SR-IOV support up to 128 VFs
- 16 GB onboard eMMC
- SPI flash / VPD EEPROM
- TruFlow configurable flow accelerator
- Hardware crypto engine
- RAID 5/6 engine
- RoCE v1/v2
- VXLAN, NVGRE, Geneve, GRE support
- UART serial console via 3.5 mm connector on bracket

## Repository Layout

```text
docs/
  datasheets/
  user-guides/
  application-notes/
  whitepapers/
hardware/
  uart/
  board-photos/
  component-identification/
software/
  drivers/
  linux/
firmware/
logs/
tools/
reverse-engineering/
homelab/
```

## Bring-up Checklist

1. Install one card in a Linux host.
2. Capture PCI enumeration:

```bash
lspci -nn | grep -i broadcom
lspci -nn | grep 14e4
lspci -vvv -s <device>
```

3. Capture kernel messages:

```bash
dmesg | grep -i -E 'broadcom|bnxt|stingray|14e4'
```

4. Connect serial console to the 3.5 mm UART port.
5. Capture boot logs from each card.
6. Identify card model and RAM size from rear label.
7. Back up eMMC/NVRAM before modifying firmware or partitions.

## Card Inventory Template

| Card | Model | Board PN | RAM | Serial | MACs | PCI IDs | State |
|------|-------|----------|-----|--------|------|---------|-------|
| 1 | TBD | TBD | TBD | TBD | TBD | TBD | Untested |
| 2 | TBD | TBD | TBD | TBD | TBD | TBD | Untested |
| 3 | TBD | TBD | TBD | TBD | TBD | TBD | Untested |
| 4 | TBD | TBD | TBD | TBD | TBD | TBD | Untested |

## Notes

Broadcom documents and firmware may be copyrighted or redistribution-restricted. Keep licensing in mind before publishing proprietary files publicly.
