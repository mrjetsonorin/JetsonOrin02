```bash
timedatectl show-timesync --all
```

* `systemd-timesyncd`: **active**
* DNS: **works**
* `Server: n/a`, `Packet count: 0`

That means **timesyncd never selected a server**, so it never even attempted sync.
This is a known failure mode when relying on distro defaults in constrained or unusual networks (robots, NAT, VLANs, captive Wi-Fi, etc.).

Adding **explicit servers** removes ambiguity and forces an attempt. This is exactly the right escalation before touching firewalls or switching daemons.

---

## Apply the drop-in 

```bash
sudo mkdir -p /etc/systemd/timesyncd.conf.d

sudo tee /etc/systemd/timesyncd.conf.d/10-marso.conf >/dev/null <<'EOF'
[Time]
NTP=0.pool.ntp.org 1.pool.ntp.org 0.fr.pool.ntp.org
FallbackNTP=ntp.ubuntu.com
EOF

sudo systemctl restart systemd-timesyncd
```

Then wait ~10–20 seconds and check:

```bash
timedatectl timesync-status
timedatectl status
```

### Success looks like

* `Server:` shows one of the pool servers
* `Packet count` increments
* `System clock synchronized: yes`

At that point:

```bash
sudo systemctl restart marso-joy-watch.service
```

The gate should clear.
