# Jetson AGX Orin – Partition & Boot Audit

**Device:** Jetson AGX Orin (T234)
**JetPack / L4T Version:** 36.4.0
**Boot Device:** NVMe
---

# 1. Physical Block Devices

## Command

```bash
lsblk -d -o NAME,MODEL,SIZE,ROTA,TYPE
```

## Result

```
mmcblk0                        58.3G disk
mmcblk0boot0                      4M disk
mmcblk0boot1                      4M disk
nvme0n1      SQF-C8MV4-1TDEDC 953.9G disk
zram0–zram7                      3.8G swap
loop devices                     snap mounts
```

**Observation:**

* eMMC present (58.3G)
* NVMe present (953.9G)
* System uses zram swap

---

# 2. Full Partition Layout (NVMe + eMMC)

## Command

```bash
lsblk -o NAME,FSTYPE,SIZE,MOUNTPOINT,PARTLABEL
```

## NVMe Layout

```
nvme0n1
├─nvme0n1p1  ext4   952.4G  /          APP
├─nvme0n1p2           128M             A_kernel
├─nvme0n1p3           768K             A_kernel-dtb
├─nvme0n1p4          31.6M             A_reserved_on_user
├─nvme0n1p5           128M             B_kernel
├─nvme0n1p6           768K             B_kernel-dtb
├─nvme0n1p7          31.6M             B_reserved_on_user
├─nvme0n1p8            80M             recovery
├─nvme0n1p9           512K             recovery-dtb
├─nvme0n1p10 vfat      64M  /boot/efi  esp
├─nvme0n1p11           80M             recovery_alt
├─nvme0n1p12          512K             recovery-dtb_alt
├─nvme0n1p13           64M             esp_alt
├─nvme0n1p14          400M             UDA
└─nvme0n1p15        479.5M             reserved
```

## eMMC Layout

```
mmcblk0
├─mmcblk0p1  ext4    56.9G             APP
├─mmcblk0p2           128M             A_kernel
├─mmcblk0p3           768K             A_kernel-dtb
├─mmcblk0p4          31.6M             A_reserved_on_user
├─mmcblk0p5           128M             B_kernel
├─mmcblk0p6           768K             B_kernel-dtb
├─mmcblk0p7          31.6M             B_reserved_on_user
├─mmcblk0p8            80M             recovery
├─mmcblk0p9           512K             recovery-dtb
├─mmcblk0p10 vfat      64M             esp
├─mmcblk0p11           80M             recovery_alt
├─mmcblk0p12          512K             recovery-dtb_alt
├─mmcblk0p13           64M             esp_alt
├─mmcblk0p14          400M             UDA
└─mmcblk0p15        479.5M             reserved
```

---

# 3. GPT Details (NVMe)

## Command

```bash
sudo gdisk -l /dev/nvme0n1
```

## Result Summary

* Disk Size: 953.9 GiB
* Partition Table: GPT
* Free Space: 31 sectors (≈15.5 KiB)

**APP partition occupies nearly entire disk (952.4 GiB).**

---

# 4. Root Filesystem Verification

## Command

```bash
findmnt /
```

## Result

```
/      /dev/nvme0n1p1 ext4
```

## Kernel Boot Parameters

```bash
cat /proc/cmdline
```

Result:

```
root=/dev/nvme0n1p1 rw rootwait rootfstype=ext4 ...
```

**Conclusion:** RootFS is on NVMe (APP partition).

---

# 5. EFI / UEFI Verification

## Command

```bash
mount | grep efi
```

## Result

```
/dev/nvme0n1p10 on /boot/efi type vfat
```

## Additional Verification

```bash
ls /sys/firmware/efi
```

Result:

```
efivars  esrt  fw_platform_size  systab
```

**Conclusion:**

* System is booting in UEFI mode
* ESP partition = nvme0n1p10 (64MB)

---

# 6. Bootloader Slot Status (A/B Enabled)

## Command

```bash
sudo nvbootctrl dump-slots-info
```

## Result

```
Current version: 36.4.0
Current bootloader slot: A
Active bootloader slot: A
num_slots: 2
slot: 0 normal
slot: 1 normal
```

## Active Slot

```bash
sudo nvbootctrl get-current-slot
```

Result:

```
0
```

**Conclusion:**

* Bootloader redundancy enabled (A/B)
* Currently booting from slot A

---

# 7. RootFS Redundancy Status

Only one APP partition exists on NVMe:

```
nvme0n1p1  APP  952.4G
```

There is no APP_b partition.

**Conclusion:**

* Bootloader A/B: ENABLED
* RootFS A/B: DISABLED

---

# 8. Boot Configuration File

## Command

```bash
cat /etc/nv_boot_control.conf
```

## Result

```
TEGRA_BOOT_STORAGE nvme0n1
TEGRA_CHIPID 0x23
```

**Conclusion:** System explicitly configured to boot from NVMe.

---

# Final Architecture Summary

| Component       | Status           |
| --------------- | ---------------- |
| Boot Device     | NVMe             |
| RootFS          | NVMe APP (952GB) |
| UEFI            | Enabled          |
| ESP Size        | 64MB             |
| Bootloader A/B  | Enabled          |
| RootFS A/B      | Disabled         |
| Disk Free Space | ~0               |

# Alba Observations
Entire NVMe is allocated to APP (root filesystem). No separate partitions for models, data, or customer storage. 64MB ESP is minimal for production firmware updates. eMMC retains full NVIDIA default layout but is unused for boot.
