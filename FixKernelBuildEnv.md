The kernel build and source symlinks under /lib/modules/5.15.148-tegra/ were originally pointing to a non-existent host-side BSP path, indicating that the kernel was built off-target. I corrected these symlinks to point to the locally installed NVIDIA kernel headers so that standard tooling can resolve build and source paths without failure.

After correction, the system is stable and running correctly; however, only header-only kernel trees are present on the target. The full kernel source tree (including the tools/ directory required for modules_prepare and full out-of-tree workflows) is not available on the device. As a result, full kernel preparation or rebuilds cannot be performed safely on-target, and kernel development remains dependent on the original Advantech BSP build environment. The attached JSON report documents the current state in a machine-readable form.







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

Expected:

```text
build  -> .../kernel-source
source -> .../kernel-source
```

---

### ✅ Clean up *only after confirmation*

```bash
sudo rm -rf /lib/modules/5.15.148-tegra/backup-links
```




NOW FULL REPORT:

```bash
cat > jetson_kernel_build_env_report.json <<EOF
{
  "report_type": "jetson_kernel_build_environment",
  "generated_at": "$(date -Is)",
  "hostname": "$(hostname)",

  "system": {
    "architecture": "$(uname -m)",
    "kernel_release": "$(uname -r)",
    "kernel_full": "$(uname -a)"
  },

  "nvidia_l4t": {
    "release_file": "$(head -n 1 /etc/nv_tegra_release 2>/dev/null || echo not_present)",
    "installed_packages": [
$(dpkg -l | awk '/nvidia-l4t-kernel/ {printf "      \"%s %s\",\n", $2, $3}')
      "sentinel_end"
    ]
  },

  "kernel_modules": {
    "modules_directory": "/lib/modules/$(uname -r)",
    "build_symlink": {
      "path": "/lib/modules/$(uname -r)/build",
      "resolved_target": "$(readlink -f /lib/modules/$(uname -r)/build 2>/dev/null || echo BROKEN)"
    },
    "source_symlink": {
      "path": "/lib/modules/$(uname -r)/source",
      "resolved_target": "$(readlink -f /lib/modules/$(uname -r)/source 2>/dev/null || echo BROKEN)"
    }
  },

  "headers": {
    "headers_installed": "$(test -d /usr/src/linux-headers-$(uname -r)-ubuntu22.04_aarch64 && echo true || echo false)",
    "headers_path": "/usr/src/linux-headers-$(uname -r)-ubuntu22.04_aarch64",
    "contains_full_kernel_source": false
  },

  "kernel_source": {
    "full_kernel_source_found_on_target": false,
    "expected_source_origin": "Advantech BSP host build environment",
    "reason": "Kernel was built off-target; only headers were shipped"
  },

  "status_summary": {
    "symlinks_corrected": true,
    "runtime_kernel_state": "healthy",
    "oot_module_build_on_target": "partially_supported",
    "full_kernel_rebuild_on_target": "not_supported"
  },

  "vendor_recommendations": [
    "Provide full kernel source tree matching running kernel",
    "Or provide official BSP-based OOT module build workflow",
    "Avoid shipping broken build/source symlinks in images"
  ]
}
EOF
```


