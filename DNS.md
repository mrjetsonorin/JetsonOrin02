# When Tailscale was managing DNS, it replaced `/etc/resolv.conf`.

When we disabled it, the file stayed static instead of restoring the systemd symlink.

That’s why DNS remained stuck.


---

Restore Ubuntu’s default DNS wiring.

Run:

```bash
sudo tailscale down
sudo systemctl restart tailscaled
sudo tailscale up --reset --accept-dns=false --ssh

sudo rm /etc/resolv.conf
sudo ln -s /run/systemd/resolve/stub-resolv.conf /etc/resolv.conf
sudo systemctl restart systemd-resolved
```

Now verify:

```bash
ls -l /etc/resolv.conf
cat /etc/resolv.conf
```

You should see:

```
/etc/resolv.conf -> /run/systemd/resolve/stub-resolv.conf
```

And inside:

```
nameserver 127.0.0.53
```

---

# 🔎 Confirm Real Upstream DNS

Run:

```bash
resolvectl status | grep "Current DNS Server"
```

You should now see only:

```
Current DNS Server: 192.168.0.1
```

(no 100.100.100.100)

---


```bash
ping google.com
```

If it resolves → fully fixed.

---
