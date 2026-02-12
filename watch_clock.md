```bash
sudo mkdir -p /etc/systemd/timesyncd.conf.d

sudo tee /etc/systemd/timesyncd.conf.d/10-marso.conf >/dev/null <<EOF
[Time]
NTP=pool.ntp.org
FallbackNTP=ntp.ubuntu.com
EOF

sudo systemctl restart systemd-timesyncd

timedatectl timesync-status
timedatectl show -p NTPSynchronized --value

sudo apt remove systemd-timesyncd -y
sudo apt install chrony -y
sudo systemctl enable chrony
sudo systemctl restart chrony
sleep 5
chronyc tracking

```

```bash
~$ chronyc sources -v

  .-- Source mode  '^' = server, '=' = peer, '#' = local clock.
 / .- Source state '*' = current best, '+' = combined, '-' = not combined,
| /             'x' = may be in error, '~' = too variable, '?' = unusable.
||                                                 .- xxxx [ yyyy ] +/- zzzz
||      Reachability register (octal) -.           |  xxxx = adjusted offset,
||      Log2(Polling interval) --.      |          |  yyyy = measured offset,
||                                \     |          |  zzzz = estimated error.
||                                 |    |           \
MS Name/IP address         Stratum Poll Reach LastRx Last sample
===============================================================================
^+ prod-ntp-3.ntp1.ps5.cano>     2   6    17    19    +63us[  +72us] +/- 6443us
^* prod-ntp-5.ntp1.ps5.cano>     2   6    17    18   +950ns[+9545ns] +/- 6440us
^+ prod-ntp-4.ntp4.ps5.cano>     2   6    17    20   -357us[ -348us] +/- 6808us
^- alphyn.canonical.com          2   6    17    19  +3143us[+3152us] +/-   64ms
^+ dns.freewebworld.fr           1   6    17    20    +87us[  +95us] +/- 6309us
^- dns-v3.ns4v.icu               2   6    17    20    +15us[  +23us] +/-   42ms
^- gofer.v4.goneco.de            2   6    17    19   +404us[ +413us] +/-   31ms
^+ 176-137-36-37.abo.bbox.fr     1   6    17    19   +121us[ +130us] +/- 6998us
ubuntu@localhost:~$ sudo chronyc makestep
200 OK
ubuntu@localhost:~$ sleep 5
chronyc tracking
Reference ID    : B97DBE3A (prod-ntp-5.ntp4.ps5.canonical.com)
Stratum         : 3
Ref time (UTC)  : Thu Feb 12 14:04:24 2026
System time     : 0.000000000 seconds slow of NTP time
Last offset     : +0.000008595 seconds
RMS offset      : 0.000008595 seconds
Frequency       : 0.000 ppm slow
Residual freq   : -19.558 ppm
Skew            : 1000000.000 ppm
Root delay      : 0.012407249 seconds
Root dispersion : 31.344129562 seconds
Update interval : 2.0 seconds
Leap status     : Normal
ubuntu@localhost:~$
```
# Marso Joystick Watcher — Minimal Install (New Robot)

## Assumptions

* OS: Ubuntu (systemd present)
* Repo already cloned:

  ```bash
  git clone https://github.com/marso-robotics/marso-stack.git
  cd marso-stack
  ```
* You have `sudo`
* ROS + deps already installed (this does **not** install ROS)

---

## 1️⃣ Install the service (one command)

```bash
sudo ./scripts/local/install_joy_watch_service.sh
```

That’s it.

What this does:

* Creates user `marso` (if missing)
* Installs watcher to `/opt/marso/joy-watch`
* Installs systemd units:

  * main service
  * failure logger
  * dry-run diagnostic + timer
* Reloads systemd
* Enables and starts everything

No manual `systemctl` typing required.

---

## 2️⃣ Verify it’s running

```bash
systemctl status marso-joy-watch.service
```

Expected:

* `Active: active (running)`
* No restart loop

Optional (logs):

```bash
journalctl -u marso-joy-watch.service -f
```

---

## 3️⃣ Verify diagnostics are armed

```bash
systemctl status marso-joy-watch-dryrun.timer
```

Expected:

* `Active: active`
* Shows next trigger time

---

## 4️⃣ (Optional) Configure environment

Config file:

```bash
sudo nano /etc/marso/joy-watch.env
```

Common knobs:

```ini
# Wait up to 30s for NTP
SYNC_WAIT_MAX_SEC=30

# Skip time check (NOT recommended)
JOY_WATCH_SKIP_NTP=0

# Optional ROS workspace setup
WS_SETUP=/opt/ros/humble/setup.bash
```

After editing:

```bash
sudo systemctl restart marso-joy-watch.service
```

---

## 5️⃣ If it fails immediately (common on new robots)

### Disable MDWX hardening (safe fallback)

```bash
sudo systemctl edit marso-joy-watch.service
```

Paste:

```ini
[Service]
MemoryDenyWriteExecute=false
```

Then:

```bash
sudo systemctl daemon-reload
sudo systemctl restart marso-joy-watch.service
```

---

## 6️⃣ Verify failure logging (once)

```bash
journalctl -t marso-joy-watch -p err
```

You should see entries **only if the service failed**.

---

## File locations (don’t memorize, just for reference)

| Thing         | Path                                          |
| ------------- | --------------------------------------------- |
| Code          | `/opt/marso/joy-watch/joy_watch.py`           |
| Runtime state | `/run/marso-joy-watch/`                       |
| Config        | `/etc/marso/joy-watch.env`                    |
| Service       | `/etc/systemd/system/marso-joy-watch.service` |

---

## Uninstall (if needed)

```bash
sudo systemctl disable --now marso-joy-watch.service
sudo systemctl disable --now marso-joy-watch-dryrun.timer
sudo rm -rf /opt/marso/joy-watch
sudo rm -f /etc/systemd/system/marso-joy-watch*
sudo systemctl daemon-reload
```

---

## Mental model (one sentence)

> “Install script owns everything; systemd keeps it alive; logs tell the truth.”
