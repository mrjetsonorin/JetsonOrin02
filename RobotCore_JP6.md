ROS2 FOLLOW:

https://docs.ros.org/en/humble/Installation/Ubuntu-Install-Debs.html

```b
sudo apt-get install -y \
  build-essential \
  cmake \
  git \
  python3-dev \
  python3-pip \
  python3-colcon-common-extensions \
  python3-rosdep


sudo rosdep init || true
rosdep update
```


VOIP bringe dependencies
```bash
sudo apt-get update
sudo apt-get install -y \
  baresip \
  baresip-core \
  libre-dev \
  libssl-dev \
  libcurl4-openssl-dev \
  uuid-dev \
  libuuid1 \
  baresip-ffmpeg \
  baresip-gstreamer

```

```bash
sudo apt-get install -y baresip baresip-core libre-dev
which baresip
```

Audio stack (VoIP media I/O)
```bash
sudo apt-get install -y \
  libportaudio2 \
  portaudio19-dev \
  pulseaudio \
  libpulse-dev

```

If headless robot, enable PulseAudio user service:
```bash
systemctl --user enable pulseaudio
systemctl --user start pulseaudio
pactl info
```

CAN bus utilities (wheels)
```bash
sudo apt-get install -y can-utils
sudo modprobe can
sudo modprobe can_raw
sudo modprobe can_dev

#(Optional persistence):
echo -e "can\ncan_raw\ncan_dev" | sudo tee /etc/modules-load.d/can.conf
```

Debugging / network tools
```bash
sudo apt-get install -y \
  wireshark \
  tcpdump \
  netcat-openbsd \
  net-tools

```
Allow non-root packet capture:
```bash
sudo dpkg-reconfigure wireshark-common
sudo usermod -aG wireshark $USER
groups | grep wireshark
```


Pyhton
```bash
pip3 install --upgrade pip
sudo apt install python3-opencv
```

Azure IoT C SDK
```bash
sudo apt-get install -y libssl-dev cmake curl uuid-dev

cd ~
git clone --recursive https://github.com/Azure/azure-iot-sdk-c.git
cd azure-iot-sdk-c
mkdir build
cd build
```

```bash
cmake -DUSE_OPENSSL=ON -DCMAKE_BUILD_TYPE=Release ..
cmake --build .
sudo cmake --install .

echo "/usr/local/lib" | sudo tee /etc/ld.so.conf.d/azure-iot.conf
sudo ldconfig
ls /usr/local/include/azureiot | head
ls /usr/local/lib | grep iothub
```

Clone marsostack
```bash
gh clone ...
cd marso-stack
git submodule update --init --recursive


cd ~/dev_am/marso-stack/marso_ws/src/control_arms/kord/kord-api-v3.0.0
mkdir -p build && cd build
cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr/local ..
make -j$(nproc)
sudo make install
```
```bash
cd ~/dev_am/marso-stack/marso_ws
source /opt/ros/humble/setup.bash
export CMAKE_PREFIX_PATH=/usr/local:$CMAKE_PREFIX_PATH
colcon build --symlink-install --cmake-args -DCMAKE_BUILD_TYPE=Release
```

Environment variables (VoIP / Azure)
Per repo rules, set these in .env (and make sure they’re exported in your shell before launch):

.env (do NOT commit):
```bash
  $ cat > /home/ubuntu/dev_am/marso-stack/.env <<'EOF'
  AZURE_IOT_BROKER_URL="mqtts://MARSO-MQTT-001.azure-devices.net:8883"
  AZURE_IOT_DEVICE_ID="Marso-001"
  AZURE_IOT_SHARED_ACCESS_KEY="IOu+bISu3NfcHXC/ay+606HCU0qt5/+CrsNly1UzbDY="
  EOF

  set -a
  source /home/ubuntu/dev_am/marso-stack/.env
  set +a

  echo "AZURE_IOT_BROKER_URL=$AZURE_IOT_BROKER_URL"
  echo "AZURE_IOT_DEVICE_ID=$AZURE_IOT_DEVICE_ID"
  if [[ -n "${AZURE_IOT_SHARED_ACCESS_KEY:-}" ]]; then echo "AZURE_IOT_SHARED_ACCESS_KEY=***set***"; else echo "AZURE_IOT_SHARED_ACCESS_KEY=***missing***"; fi
```



LAUNCH
```bash
  /home/ubuntu/dev_am/marso-stack/scripts/teleop_local.sh
```

  It will:

  - load .env
  - source ROS2 + workspace
  - run preflight checks
  - prompt if warnings
  - launch teleop.launch.py with defaults on
