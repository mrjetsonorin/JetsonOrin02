  # system deps
  sudo apt-get update
  sudo apt-get install -y git cmake pkg-config python3 autoconf automake libtool zlib1g-dev

  # build Parrot libs using Alchemy
  WORK_DIR=/tmp/parrot-telemetry-work
  SRC_ROOT=/home/ubuntu/dev_am/marso-stack/telemetry/vendor/parrot
  ALCHEMY_DIR="${WORK_DIR}/alchemy"
  TARGET_OUT="${WORK_DIR}/out"

  rm -rf "${WORK_DIR}"
  mkdir -p "${WORK_DIR}"
  git clone --depth 1 https://github.com/Parrot-Developers/alchemy "${ALCHEMY_DIR}"

  export TARGET_PRODUCT="pclinux"
  export TARGET_CONFIG_DIR="${WORK_DIR}/config"
  export TARGET_OUT="${TARGET_OUT}"
  export TARGET_OS="linux"
  export TARGET_OS_FLAVOUR="native"
  export TARGET_LIBC="native"
  export TARGET_ARCH="$(uname -m)"
  export TARGET_GLOBAL_CFLAGS="-fPIC"
  export ALCHEMY_WORKSPACE_DIR="${WORK_DIR}"
  export TARGET_SCAN_ADD_DIRS="${SRC_ROOT}"
  export TARGET_SCAN_PRUNE_DIRS="${SRC_ROOT}/../parrot-telemetry"
  export ALCHEMY_USE_COLORS=0

  python3 "${ALCHEMY_DIR}/scripts/alchemake.py" -f "${ALCHEMY_DIR}/main.mk" libtelemetry libpomp

  # install to /usr/local
  STAGING_DIR="${TARGET_OUT}/staging"
  if [ -d "${STAGING_DIR}/usr" ]; then
    sudo cp -a "${STAGING_DIR}/usr/." /usr/local/
  else
    sudo cp -a "${STAGING_DIR}/." /usr/local/
  fi
  sudo mkdir -p /usr/local/include
  sudo cp -a "${SRC_ROOT}/telemetry/libtelemetry/include/." /usr/local/include/
  sudo cp -a "${SRC_ROOT}/libshdata_stub/include/." /usr/local/include/
  sudo cp -a "${SRC_ROOT}/libpomp/include/." /usr/local/include/
  sudo ldconfig

  Then rebuild:

  cd /home/ubuntu/dev_am/marso-stack/marso_ws
  colcon build
