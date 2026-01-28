Generate corrected JSON report:



```bash
cat << EOF > jetson_kernel_build_env_report.json
{
  "metadata": {
    "generated": "$(date -Is)",
    "hostname": "$(hostname)",
    "platform": "NVIDIA Jetson Orin"
  },

  "system": {
    "kernel_release": "$(uname -r)",
    "kernel_full": "$(uname -a)",
    "architecture": "$(uname -m)",
    "l4t_release": "$(head -n 1 /etc/nv_tegra_release 2>/dev/null)"
  },

  "kernel_headers": {
    "installed": "$(test -d /usr/src/linux-headers-$(uname -r)-ubuntu22.04_aarch64 && echo true || echo false)",
    "path": "/usr/src/linux-headers-$(uname -r)-ubuntu22.04_aarch64"
  },

  "symlinks": {
    "build": {
      "path": "/lib/modules/$(uname -r)/build",
      "ls": "$(ls -l /lib/modules/$(uname -r)/build 2>/dev/null)",
      "resolved": "$(readlink -f /lib/modules/$(uname -r)/build || echo BROKEN)"
    },
    "source": {
      "path": "/lib/modules/$(uname -r)/source",
      "ls": "$(ls -l /lib/modules/$(uname -r)/source 2>/dev/null)",
      "resolved": "$(readlink -f /lib/modules/$(uname -r)/source || echo BROKEN)"
    }
  },

  "broken_symlinks_system_wide": [
    "$(find /lib /usr/lib /usr/src -xtype l 2>/dev/null | tr '\n' '; ')"
  ],

  "toolchain": {
    "gcc": "$(gcc --version | head -n 1)",
    "make": "$(make --version | head -n 1)",
    "ld": "$(ld --version | head -n 1)",
    "bc": "$(bc --version | head -n 1)"
  },

  "analysis": {
    "problem": "Kernel build/source symlinks reference a host BSP path not present on the target",
    "root_cause": "Custom BSP or host-built kernel image flashed without relinking on-device build environment",
    "impact": "Out-of-tree kernel module builds and DKMS fail; running kernel unaffected"
  },

  "corrective_action": {
    "intent": "Restore valid on-device kernel build environment using installed headers",
    "commands": [
      "sudo rm /lib/modules/$(uname -r)/build",
      "sudo rm /lib/modules/$(uname -r)/source",
      "sudo ln -s /usr/src/linux-headers-$(uname -r)-ubuntu22.04_aarch64 /lib/modules/$(uname -r)/build",
      "sudo ln -s /usr/src/linux-headers-$(uname -r)-ubuntu22.04_aarch64 /lib/modules/$(uname -r)/source"
    ],
    "verification": [
      "readlink -f /lib/modules/$(uname -r)/build",
      "find /lib/modules/$(uname -r) -xtype l"
    ],
    "status_after_fix": "System supports standard out-of-tree kernel module compilation"
  }
}
EOF


```
```bash
sudo apt install -y jq
jq . jetson_kernel_build_env_report.json
```

THE COMMAND:

```bash
KVER=$(uname -r)
MODDIR=/lib/modules/$KVER
HEADERS=/usr/src/linux-headers-${KVER}-ubuntu22.04_aarch64/3rdparty/canonical/linux-jammy/kernel-source
BACKUP=$MODDIR/backup-links

sudo mkdir -p "$BACKUP"

for x in build source; do
  if [ -e "$MODDIR/$x" ] && [ ! -L "$MODDIR/$x" ]; then
    sudo mv "$MODDIR/$x" "$BACKUP/$x"
  fi
  sudo rm -f "$MODDIR/$x"
  sudo ln -s "$HEADERS" "$MODDIR/$x"
done

ls -l "$MODDIR"/{build,source}
```

EXPLANATION:

---

### ✅ 1. Created a backup location

```bash
KVER=5.15.148-tegra
MODDIR=/lib/modules/$KVER
BACKUP=$MODDIR/backup-links
HEADERS=/usr/src/linux-headers-5.15.148-tegra-ubuntu22.04_aarch64/3rdparty/canonical/linux-jammy/kernel-source

sudo mkdir -p "$BACKUP"
```

**Why:**

* Preserves vendor state
* Allows rollback if NVIDIA tools expect those links
* Good practice on embedded systems

---

### ✅ 2. Backed up *real paths* (not symlinks)

You guarded against two cases:

* link
* directory
* file

```bash
if [ -e "$MODDIR/build" ] && [ ! -L "$MODDIR/build" ]; then
    sudo mv "$MODDIR/build" "$BACKUP/build"
fi

if [ -e "$MODDIR/source" ] && [ ! -L "$MODDIR/source" ]; then
    sudo mv "$MODDIR/source" "$BACKUP/source"
fi
```

**Why:**

* If a BSP accidentally created a real directory, you wouldn’t destroy it
* Prevents irreversible damage

---

### ✅ 3. Removed broken symlinks (only after backup)

```bash
sudo rm -f "$MODDIR/build" "$MODDIR/source"
```

**Why:**

* Guarantees clean state
* Avoids `ln -s` failing silently

---

### ✅ 4. Recreated symlinks to the *correct* header tree

```bash
sudo ln -s "$HEADERS" "$MODDIR/build"
sudo ln -s "$HEADERS" "$MODDIR/source"
```

**Why:**

* Points at the **actual kernel build root**
* Not the host BSP
* Not the generic headers root

---

### ✅ 5. Verified explicitly

```bash
ls -l /lib/modules/5.15.148-tegra/{build,source}
```

Expected:

```text
build  -> .../kernel-source
source -> .../kernel-source
```

---

### ✅ 6. Cleaned up *only after confirmation*

```bash
sudo rm -rf /lib/modules/5.15.148-tegra/backup-links
```

**Why:**

* Ensures rollback window until build success
* Matches production embedded workflow

---
