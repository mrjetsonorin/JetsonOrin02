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
