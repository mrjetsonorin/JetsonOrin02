```bash
timedatectl show-timesync --all
```

systemd-timesyncd is running, DNS
  works, but it has no active NTP server selected (Server: n/a,
  Packet count: 0). That means it never even tried to sync.

  What I see on your box:

  - timesyncd is active
  - timedatectl timesync-status shows no server
  - DNS resolves *.pool.ntp.org fine

  So the failure is not DNS; it’s either:

  1. No NTP servers configured/selected, or
  2. UDP/123 is blocked outbound, so it never reaches the servers.

  ### Fix (best practice)

  Add explicit servers via a drop‑in and restart timesyncd:
```bash

  sudo mkdir -p /etc/systemd/timesyncd.conf.d
  sudo tee /etc/systemd/timesyncd.conf.d/10-marso.conf >/dev/null
  <<'EOF'
  [Time]
  NTP=0.pool.ntp.org 1.pool.ntp.org 0.fr.pool.ntp.org
  FallbackNTP=ntp.ubuntu.com
  EOF
```
```bash

  sudo systemctl restart systemd-timesyncd
  timedatectl timesync-status
```
  If it still shows Server: n/a and Packet count: 0, then UDP/123
  is blocked by network/firewall. We’ll need to open that.

  Want me to apply the drop‑in now?
