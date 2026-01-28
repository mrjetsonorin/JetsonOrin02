```bash
sudo apt update && sudo apt install -y build-essential curl git vim nano wget gh tree tmux htop
git --version
gh --version
sudo apt upgrade
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

Install firefox:
```bash


```

```bash
sudo apt install -y python3-pip python3-venv python3-dev
python3 -m pip install --upgrade pip
python3 --version
sudo reboot
```



