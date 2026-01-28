Check first the correct the kernels.md.


```bash
ls -l /lib/modules/5.15.148-tegra/{build,source}
```
Expected output:
```bash

build  -> /usr/src/linux-headers-5.15.148-tegra-ubuntu22.04_aarch64/3rdparty/canonical/linux-jammy/kernel-source
source -> /usr/src/linux-headers-5.15.148-tegra-ubuntu22.04_aarch64/3rdparty/canonical/linux-jammy/kernel-source
```
  If everything works and you want to clean up the backup:

```bash
  sudo rm -rf /lib/modules/5.15.148-tegra/backup-links

```
Running kernel: 5.15.148-tegra-36.4.0

APT candidate headers: 36.4.7

NVIDIA ties kernel ABI + headers + BSP tightly.

Building out-of-tree modules with mismatched headers is not safe (and your guard correctly stopped it).

So you must realign them.
```bash
sudo apt-get install -y \
  nvidia-l4t-kernel=5.15.148-tegra-36.4.0-20240912212859 \
  nvidia-l4t-kernel-headers=5.15.148-tegra-36.4.0-20240912212859
sudo apt-mark hold nvidia-l4t-kernel nvidia-l4t-kernel-headers
uname -r
ls -l /lib/modules/$(uname -r)/build
```
Expected:
```bash
uname -r still shows 5.15.148-tegra

/lib/modules/5.15.148-tegra/build exists and contains a Makefile
```
If that’s true, proceed.

```bash
mkdir -p ~/scripts
wget https://linux.brostrend.com/install -O ~/scripts/brostrend-install.sh
cat ~/scripts/brostrend-install.sh
```


Run them with sudo:
```bash
#  - Install:
sudo /home/ubuntu/rtl8852bu-install

# Run from the source directory:

ls -lh /lib/modules/$(uname -r)/extra/rtl8852bu/8852bu.ko
lsmod | grep -i 8852
sudo dmesg | tail -n 50
nmcli dev status


#  - Uninstall if broken:

sudo /home/ubuntu/rtl8852bu-uninstall
# Then :

sudo reboot
lsmod | grep 8852
nmcli dev status




```
