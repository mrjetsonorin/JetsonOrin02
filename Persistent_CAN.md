```bash
ls -l /etc/systemd/network/
```
OUTPUT: `-rw-r--r-- 1 root root 56 Sep 13  2024 10-jetson-onboard-ethernet.link`

---

## Create CAN0 config (250 kbit/s)

```bash
sudo nano /etc/systemd/network/10-can0.network
```

Paste:

```
[Match]
Name=can0

[CAN]
BitRate=250000
```

Save:* `Ctrl + O`* `Enter`* `Ctrl + X`

---

## Create CAN1 config (500 kbit/s)

```bash
sudo nano /etc/systemd/network/10-can1.network
```

```
[Match]
Name=can1

[CAN]
BitRate=500000
ListenOnly=no
RestartSec=0.1
```
---

## Apply the configuration

```bash
sudo systemctl restart systemd-networkd
```
```bash
sudo reboot
```
```bash
ip -details link show can0 | grep bitrate
ip -details link show can1 | grep bitrate
```

You should see:

* `can0` → `250000`
* `can1` → `500000`

---

## Test each bus

```bash
candump can0
```
YOU WILL SEE READINGS...

```bash
candump can1 &
cansend can1 123#DEADBEEF
```

Seeing:
```
can1  123   [4]  DE AD BE EF
```
means:

* kernel driver OK
* mttcan OK
* transceiver powered
* CANH/CANL loopback works
* bitrate is accepted

Silence after that is **EXPECTED** if nothing else talks, i.e. no wheel movement commands.
