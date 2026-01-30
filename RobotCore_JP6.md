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



  1. Rebuild so the updated env_utils.py is used by launch:

  cd /home/ubuntu/dev_am/marso-stack/marso_ws
  source /opt/ros/humble/setup.bash
  colcon build --symlink-install --packages-select marso_bringup

  2. Validate device ID now shows Marso-001:

  /home/ubuntu/dev_am/marso-stack/scripts/teleop_local.sh

  3. Run local healthcheck when needed:

  /home/ubuntu/dev_am/marso-stack/scripts/marso_healthcheck_local.sh

  If you want, I can wire the healthcheck to run automatically after launch (with a short grace period) and print a concise PASS/FAIL summary.








   1) Install missing dependency (needs sudo):

  sudo apt-get update
  sudo apt-get install -y ros-humble-backward-ros

  2) Install ZED wrapper deps + build just ZED packages:

  cd /home/ubuntu/dev_am/marso-stack/marso_ws
  source /opt/ros/humble/setup.bash
  rosdep install --from-paths src/zed-ros2-wrapper -y --ignore-src
  colcon build --symlink-install --packages-up-to zed_wrapper --cmake-args=-DCMAKE_BUILD_TYPE=Release

  (These are the same steps from Stereolabs’ ROS2 guide.) (stereolabs.com (https://www.stereolabs.com/docs/ros2?utm_source=openai))

  3) Launch the ZED camera node (ZED‑X):

  ros2 launch zed_wrapper zed_camera.launch.py camera_model:=zedx

  (stereolabs.com (https://www.stereolabs.com/docs/ros2?utm_source=openai))

  4) Wire it into your teleop launch:

  export CAMERA_LAUNCH=/home/ubuntu/dev_am/marso-stack/marso_ws/install/zed_wrapper/share/zed_wrapper/launch/zed_camera.launch.py
  export CAMERA_MODEL=zedx
  /home/ubuntu/dev_am/marso-stack/scripts/teleop_local.sh

  Once that’s done, the “no camera publishers” warning should go away.



  source /home/ubuntu/dev_am/marso-stack/marso_ws/install/setup.bash
  ros2 launch zed_wrapper zed_camera.launch.py camera_model:=zedx

  If you want it persistent:

  echo "source /home/ubuntu/dev_am/marso-stack/marso_ws/install/setup.bash" >> ~/.bashrc

  Then retry the launch.
