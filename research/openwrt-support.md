# OpenWRT Support Investigation

## Status

Confirmed

## Evidence

### Kernel Image

openwrt-brcmiproc-arm64-broadcom_ps225-hxx-Image

### Root Filesystem

openwrt-brcmiproc-arm64-broadcom_ps225-hxx-rootfs.ext4.gz

### Device Tree

openwrt-brcmiproc-arm64-broadcom_ps225-hxx-bcm958802a802x.dtb

## DTB Findings

### Platform

- Broadcom Stingray
- BCM958802A802x

### CPU

- 8x ARM Cortex-A72

### Memory

- DDR4 memory subsystem present

### UART

- Multiple UART controllers
- 115200 console configuration

### SPI

- Multiple SPI controllers

## Significance

Evidence has been found of a dedicated OpenWRT build target for the
Broadcom PS225 platform:

- openwrt-brcmiproc-arm64-broadcom_ps225-hxx-Image
- openwrt-brcmiproc-arm64-broadcom_ps225-hxx-rootfs.ext4.gz
- openwrt-brcmiproc-arm64-broadcom_ps225-hxx-bcm958802a802x.dtb

This moves ARM-side operating system support from
"theoretical" to "demonstrated".
