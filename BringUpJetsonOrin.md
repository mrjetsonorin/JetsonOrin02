```bash
sudo apt update && sudo apt install -y build-essential curl vim nano wget tree tmux htop snapd
sudo apt upgrade
sudo systemctl enable --now snapd
sudo systemctl enable --now snapd.socket
```


```bash
echo 'export PATH=/usr/local/cuda/bin:$PATH' >> ~/.bashrc
echo 'export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH' >> ~/.bashrc
source ~/.bashrc

nvcc --version
```

```bash
# sudo nvpmodel -m <mode_id>  # e.g., MAXN, 30W, custom XML
sudo nvpmodel -m 0  # MAXN (full power)

sudo jetson_clocks
cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_min_freq  # see min freq
cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_max_freq  # see max freq
sudo cat /sys/devices/system/cpu/cpu*/cpufreq/scaling_cur_freq  # see current
```


LOCK:
```bash
# View held packages
apt-mark showhold

# Lock NVIDIA stack for production
sudo apt-mark hold 'nvidia-*' 'libcudnn*' 'tensorrt*' 'cuda-*' 'libcuda*'

# For flexible development (lighter lock)
sudo apt-mark hold 'nvidia-l4t-*' 'tensorrt*' 'libcudnn*'

# View held packages
apt-mark showhold

sudo apt update
sudo apt -y upgrade     # Upgrade existing packages

sudo apt autoremove 
sudo apt autoclean

sudo reboot
```

```bash
sudo apt install -y python3-pip python3-venv python3-dev pip
python3 -m pip install --upgrade pip
python3 --version
```

```bash
#sudo apt update
sudo pip install jetson-stats # or sudo pip install -U jetson-stats  ?
sudo reboot
jtop
```


Install firefox:
```bash
sudo snap remove firefox 2>/dev/null || true && \
sudo add-apt-repository -y ppa:mozillateam/ppa && \
sudo apt update && \
sudo tee /etc/apt/preferences.d/firefox-no-snap >/dev/null <<'EOF'
Package: firefox*
Pin: release o=LP-PPA-mozillateam
Pin-Priority: 1001
EOF
sudo apt install -y firefox && \
which firefox && firefox --version
```


Install and configure SSH

```bash
sudo apt install -y openssh-server
sudo systemctl enable --now ssh
systemctl status ssh
```

```bash
curl -fsSL https://tailscale.com/install.sh | sh
```

```bash
sudo tailscale up
```

Verify status and IP:

```bash
tailscale status
tailscale ip -4
```

```bash
git config --global user.name "mrjetsonorin"
git config --global user.email "mrjorin00@email.com"
git config --global init.defaultBranch main
git config --global pull.rebase false
git config --global pull.ff only           # forbid accidental merge commits
git config --global fetch.prune true       # auto-prune deleted remote branches
git config --global rebase.autoStash true  # stash local changes during rebase
git config --global core.autocrlf input    # safer line endings for Windows + WSL
git config --global rerere.enabled true    # remembers conflict resolutions
git config --global -l
```
```bash
type -p curl >/dev/null || sudo apt-get update && sudo apt-get install -y curl
curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg
sudo chmod go+r /usr/share/keyrings/githubcli-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null
sudo apt update && sudo apt install -y gh
```

```bash
gh auth login
gh auth status
```


uv, npm..
```bash
sudo apt install npm
```


if problems with gh auth:
asra702a1:~/azure-iot-sdk-c/build/build/build$ gh auth login
? Where do you use GitHub? GitHub.com
? What is your preferred protocol for Git operations on this host? HTTPS
? Authenticate Git with your GitHub credentials? Yes
? How would you like to authenticate GitHub CLI? Login with a web browser

! First copy your one-time code: 9431-7463
Press Enter to open https://github.com/login/device in your browser...
[49424] Sandbox: CanCreateUserNamespace() unshare(CLONE_NEWPID): EPERM
Error: no DISPLAY environment variable specified
✓ Authentication complete.
- gh config set -h github.com git_protocol https
✓ Configured git protocol
mkdir /home/ubuntu/.config/gh: permission denied
ubuntu@nvthorasra702a1:~/azure-iot-sdk-c/build/build/build$ echo $HOME
ls -ld $HOME
/home/ubuntu
drwxr-x--- 22 ubuntu ubuntu 4096 Feb 17 11:19 /home/ubuntu
ubuntu@nvthorasra702a1:~/azure-iot-sdk-c/build/build/build$ ls -ld /home/ubuntu/.config
drwxr-xr-x 3 root root 4096 Dec 17 10:32 /home/ubuntu/.config
ubuntu@nvthorasra702a1:~/azure-iot-sdk-c/build/build/build$ ls -la /home/ubuntu/.config
total 12
drwxr-xr-x  3 root   root   4096 Dec 17 10:32 .
drwxr-x--- 22 ubuntu ubuntu 4096 Feb 17 11:19 ..
drwxr-xr-x  2 root   root   4096 Dec 17 10:32 autostart
ubuntu@nvthorasra702a1:~/azure-iot-sdk-c/build/build/build$ sudo chown -R ubuntu:ubuntu /home/ubuntu/.config
ubuntu@nvthorasra702a1:~/azure-iot-sdk-c/build/build/build$ ls -ld /home/ubuntu/.config
drwxr-xr-x 3 ubuntu ubuntu 4096 Dec 17 10:32 /home/ubuntu/.config
ubuntu@nvthorasra702a1:~/azure-iot-sdk-c/build/build/build$ gh auth status
You are not logged into any GitHub hosts. To log in, run: gh auth login
ubuntu@nvthorasra702a1:~/azure-iot-sdk-c/build/build/build$
ubuntu@nvthorasra702a1:~/azure-iot-sdk-c/build/build/build$
