STOP!


Building out-of-tree modules with mismatched headers is not safe. So you must realign them.


Check first the correct the kernels.md. The running kernel (5.15.148-tegra, built Sep 23 2025) is a vendor-built BSP kernel and does not correspond to any NVIDIA APT kernel package. Although kernel headers are installed, they originate from NVIDIA L4T 36.4.x and are ABI-incompatible with the running kernel image. This causes partial kernel trees, missing host tools, and prevents safe out-of-tree module builds. Correcting symlinks does not resolve this. A matching kernel source tree or vendor-supported OOT module build environment is required. HOWEVER, LETS MOVE OVER ALL CAUSE WE DONT SURRENDER, THIS IS NOT SAFE AT ALL, IM GONNA MIX 2 mutually exclusive worlds:

World A: NVIDIA stock kernel (APT-managed, safe to realign)

World B: Advantech BSP kernel (vendor-built, APT realignment is unsafe)

WE ARE IN WORLD B, SO IM PROPOSING A DANGEROUS THING, THOUGH IT WORKS, 
You are running Jetson Orin, L4T R36.4

Kernel version string:

5.15.148-tegra


NVIDIA encodes ABI compatibility in the package version, not the uname -r string:

5.15.148-tegra-36.4.0-<build-id>

The mismatch problem

Running kernel: built from 36.4.0

APT candidate headers: 36.4.7

Even though uname -r matches, kernel headers are NOT ABI-compatible

NVIDIA’s kernel Makefiles detect this and correctly abort module builds

That’s why your out-of-tree module build stopped.



Running kernel: 5.15.148-tegra-36.4.0

APT candidate headers: 36.4.7

NVIDIA ties kernel ABI + headers + BSP tightly.

You are NOT:

rebuilding the kernel

patching kernel source

touching /boot/Image manually

changing uname -r

You ARE:

forcing package-level ABI alignment

swapping headers + OOT headers to match the running kernel

preventing APT from silently drifting

That is already the smallest possible change that fixes the real problem.

So yes — this is a minimal rewrite.

Why it feels wrong (but isn’t)

On normal Ubuntu:

headers float independently

DKMS papers over ABI drift

On Jetson:

kernel ABI = BSP release

headers are not fungible

NVIDIA kernel is not DKMS-first

So your intuition (“this feels heavy-handed”) is valid — but the platform is the reason, not your approach.





MY BEST ON TARGET OPTION: ****Pin kernel + headers to the running BSP****
Pros

Minimal change

Fully on-target

Reversible

Proven to work

Cons

APT friction

Needs holds

➡️ This is the best option if you must build on the device


```text
uname -r
→ 5.15.148-tegra

/etc/nv_tegra_release
→ R36.4.0
→ KERNEL_VARIANT: oot
```

This tells us:

1. **Running kernel ABI**

   * `5.15.148-tegra` ✔

2. **JetPack / L4T release**

   * R36.4.0 ✔

3. **Kernel variant = OOT**

   * This explicitly means NVIDIA expects **out-of-tree modules** to be built
   * That’s exactly your use case (Wi-Fi driver)

4. **Your kernel image is already correct**

   * No evidence of 36.4.7 kernel running
   * No mismatch between `/boot/Image`, `uname`, and L4T release

So the *only* thing that can be wrong is:

> headers drifting to a newer minor release via APT

```bash
sudo apt-get install -y \
  nvidia-l4t-kernel-headers=5.15.148-tegra-36.4.0-20240912212859 \
  nvidia-l4t-kernel-oot-headers=5.15.148-tegra-36.4.0-20240912212859
```

Then freeze them (important):

```bash
sudo apt-mark hold \
  nvidia-l4t-kernel-headers \
  nvidia-l4t-kernel-oot-headers
```

```bash
ls -l /lib/modules/$(uname -r)/build
ls /lib/modules/$(uname -r)/build/Makefile
```

### Expected (GOOD):

* `..... build` → points into `....linux-headers-5.15.148-tegra-ubuntu22.04_aarch64/...`
* `Makefile` exists

If that’s true:
👉 **OOT module builds are supported**
👉 **Kernel reinstall is NOT required**
👉 **No reboot required**

---



### ✅ When you want to build an OOT module (Wi-Fi, driver, etc.)

Just do this from the driver source directory:

```bash
make -C /lib/modules/$(uname -r)/build M=$PWD modules
sudo make -C /lib/modules/$(uname -r)/build M=$PWD modules_install
sudo depmod -a
```

That’s it. No reinstall. No scripts.

---

### ❗ When you run system updates

```bash
sudo apt upgrade
```

Safe **because**:

* kernel headers are held
* ABI stays stable

---

### ⚠️ Only if you intentionally upgrade JetPack / L4T later

Then you must:

1. Unhold packages
2. Upgrade kernel + headers together
3. Re-hold them
4. Rebuild OOT modules

But **only** when you *want* a new BSP.
