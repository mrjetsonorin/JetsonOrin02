```bash
# 1. Check if video devices exist now
ls -la /dev/video*

# 2. Try opening a camera with ZED SDK tools
/usr/local/zed/tools/ZED_Explorer

# 3. Or try the depth viewer
/usr/local/zed/tools/ZED_Depth_Viewer

# 4. Check daemon logs for any errors
sudo journalctl -u zed_x_daemon -n 50
```

If cameras still don't open in the tools, try:
```bash
# Restart the ZED-X daemon
sudo systemctl restart zed_x_daemon

# Then check status
sudo systemctl status zed_x_daemon
```
