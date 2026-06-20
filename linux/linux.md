# Linux on the Stingrays

This is how I installed Arch Linux ARM to both the PS225 and PS1100R. I created the config file by extracting `/proc/config.gz` from the stock system and manually comparing it with a new config from ALARM's ``linux-aarch64``. 

## Root filesystem 

Create an ext4 filesystem on /dev/mmcblk0p5 (create the partition with fdisk if it doesn't exist yet), mount it, and extract an ALARM aarch64 generic rootfs tarball there:  
```
bsdtar -xpf ArchLinuxARM-aarch64-latest.tar.gz -C /mnt/alarm
```

## Kernel 

The `config` file can be applied to Arch Linux ARM's `linux-aarch64` package to build an up-to-date kernel image and modules. A dtb for the PS225, newer than the stock one, builds at `arch/arm64/boot/dts/broadcom/stingray/bcm958802a802x.dtb`.

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
cp /wherever/you/put/the/new/config src/config
updpkgsums
makepkg
```

From the stock Yocto environment, copy the kernel and header packages that makepkg made into the rootfs, chroot, do `mount -t proc proc /proc`, and install the packages with `pacman -U`.

On the PS225 I had to blacklist this module to keep udev from hanging the system:  
`/etc/modprobe.d/no-bcm_sba_raid.conf`  
`blacklist bcm_sba_raid`

## Bootloader 

Mount the EFI partition (/dev/mmcblk0p1) and copy the kernel image and dtb (if needed):
```
cp /boot/Image /mnt/efi/Image.2
cp /boot/dtbs/broadcom/stingray/bcm958802a802x.dtb /mnt/efi/dt-blob.bin.2
```
If `Image.2` exists, copy to `Image.3` instead and change the number in the bootloader option below.  
To boot into ALARM, interrupt the bootloader at the countdown, select the new kernel, dtb, and root partition, and start:
```
select kernel 2
select dtb 2
select rootfs-ordinal 5
startup
```
If using a PS1100R, use the stock dtb on the partiton and don't copy a new one.  
If you build a newer kernel later you must remember to copy the kernel image to the EFI partition and replace `Image.2`.

After the kernel messages you should get an ALARM login prompt on the serial console. If it boots successfully the bootloader will remember your choice as default for the next boot. The SFP28 ports on the PS225 should work without additional fuss thanks to the ``bnxt`` modules and you can use a network manager of your choice to configure them. The 100Gbps QSFP28 connector on the PS1100R is untested but should work.
