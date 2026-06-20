== Linux on the Stingrays ==

This is how I installed Arch Linux ARM to both the PS225 and PS1100R.

=== Root filesystem ===

Create an ext4 filesystem on /dev/mmcblk0p5 (create the partition with fdisk if it doesn't exist yet), mount it, and extract an ALARM aarch64 generic rootfs tarball there:  
```
bsdtar -xpf ArchLinuxARM-aarch64-latest.tar.gz -C /mnt/alarm
```

=== Kernel ===

The `config` file can be applied to Arch Linux ARM's `linux-aarch64` package to build an up-to-date kernel image and modules. A dtb for the PS225, newer than the preinstalled one, builds at `arch/arm64/boot/dts/broadcom/stingray/bcm958802a802x.dtb`.

On an existing ALARM system:
```
git clone --no-checkout https://github.com/archlinuxarm/PKGBUILDs
git sparse-checkout init
git sparse-checkout set core/linux-aarch64
git checkout master
cd PKGBUILDs/core/linux-aarch64
sed -i s/linux-aarch64/linux-aarch64-stingray/g PKGBUILD
sed -i '/_package-chromebook/,+30d' PKGBUILD
sed -i 's/ "${pkgbase}-chromebook"//' PKGBUILD
makepkg -o
cp /wherever/you/put/the/new/config src/.config
updpkgsums
makepkg
```

From the stock Yocto environment, chroot into the rootfs, do `mount -t proc proc /proc`, and install the kernel and header packages that makepkg made with `pacman -U`.

On the PS225 I had to make blacklist this module to keep dbus from hanging the system: 
`/etc/modprobe.d/no-bcm_sba_raid.conf`
```
blacklist bcm_sba_raid
```

=== Bootloader ===

Mount the EFI partition (`/dev/mmcblk0p1`) and copy the kernel image and dtb (if needed):
```
cp /boot/Image /mnt/efi/Image.2
cp /boot/dtbs/broadcom/stingray/bcm958802a802x.dtb /mnt/efi/dt-blob.bin.2
```
To boot into ALARM, interrupt the bootloader at the countdown, select the new kernel, dtb, and root partition, and start:
```
select kernel 2
select dtb 2
select rootfs-ordinal 5`
startup
```

After the kernel messages you should get an ALARM login prompt on the serial console.
