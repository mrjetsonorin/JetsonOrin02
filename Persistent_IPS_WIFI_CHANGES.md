```bash
cat >/tmp/lock_arm_net.sh <<'BASH'
#!/usr/bin/env bash
set -euo pipefail

IF_KR1205="enP5p4s0"
IF_KR1240="enP5p5s0"

IP_KR1205_LOCAL="192.168.41.51/24"
IP_KR1240_LOCAL="192.168.42.51/24"

# Remove conflicting ethernet profiles on those two NICs
while IFS=: read -r name uuid; do
    type="$(nmcli -g connection.type connection show "$uuid" 2>/dev/null || true)"
    iface="$(nmcli -g connection.interface-name connection show "$uuid" 2>/dev/null || true)"

    [[ "$type" == "802-3-ethernet" ]] || continue
    [[ "$iface" == "$IF_KR1205" || "$iface" == "$IF_KR1240" ]] || continue
    [[ "$name" == "arm-kr1205" || "$name" == "arm-kr1240" ]] && continue

    nmcli connection delete "$uuid" >/dev/null 2>&1 || true
done < <(nmcli -t -f NAME,UUID connection show)

# Remove old profiles if they exist
nmcli connection delete arm-kr1205 >/dev/null 2>&1 || true
nmcli connection delete arm-kr1240 >/dev/null 2>&1 || true

# Create KR1205 profile
nmcli connection add type ethernet ifname "$IF_KR1205" con-name arm-kr1205 \
    ipv4.method manual ipv4.addresses "$IP_KR1205_LOCAL" \
    ipv4.never-default yes ipv6.method ignore \
    connection.autoconnect yes connection.autoconnect-priority 200 >/dev/null

# Create KR1240 profile
nmcli connection add type ethernet ifname "$IF_KR1240" con-name arm-kr1240 \
    ipv4.method manual ipv4.addresses "$IP_KR1240_LOCAL" \
    ipv4.never-default yes ipv6.method ignore \
    connection.autoconnect yes connection.autoconnect-priority 200 >/dev/null

# Bring connections up
nmcli connection up arm-kr1205 >/dev/null
nmcli connection up arm-kr1240 >/dev/null

echo "Configured interfaces:"
nmcli -t -f DEVICE,STATE,CONNECTION device status | grep -E "$IF_KR1205|$IF_KR1240" || true

ip -4 addr show "$IF_KR1205" | sed -n '1,4p'
ip -4 addr show "$IF_KR1240" | sed -n '1,4p'

BASH

chmod +x /tmp/lock_arm_net.sh
sudo /tmp/lock_arm_net.sh
```

---

## Verification Commands

```bash
ping -I enP5p4s0 -c 1 -W 1 192.168.41.1
ping -I enP5p5s0 -c 1 -W 1 192.168.42.2

nc -vz -w 1 192.168.41.1 7581
nc -vz -w 1 192.168.42.2 7582
```

---

## 🔁 If Ports Are Physically Swapped

```bash
sudo nmcli con mod arm-kr1205 connection.interface-name enP5p5s0
sudo nmcli con mod arm-kr1240 connection.interface-name enP5p4s0
sudo nmcli con up arm-kr1205
sudo nmcli con up arm-kr1240
```
