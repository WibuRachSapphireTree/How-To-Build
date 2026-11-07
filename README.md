# Building Evolution X for Redmi Note 13 4G (sapphire / sapphiren)

Build guide for compiling Evolution X (Android 17 / `cnb` branch) on the Xiaomi Redmi Note 13 4G series (Snapdragon 685 - SM6225).

---

## Requirements

### Host OS
**Ubuntu 24.04 LTS (Noble Numbat)** is strongly recommended. 
Android 17 build chains rely heavily on updated Clang/LLVM toolchains, Python 3.12, and recent OpenSSL libraries. Ubuntu 24.04 provides these natively without needing deprecated PPA workarounds.

### Hardware
* **CPU:** 8 cores / 16 threads or better.
* **RAM:** 16 GB minimum + at least 32 GB swap. A17 metalava and ninja linking stages are memory-hungry and will crash if you run out of memory. 32 GB RAM is ideal.
* **Storage:** 350 GB+ available on an SSD or NVMe drive. Building on an HDD is not recommended.
* **Network:** Fast and stable connection (manifest + trees + git-lfs pull over 150 GB).

---

## 1. Environment Setup

Update your package index and install the required dependencies:

```bash
sudo apt update && sudo apt upgrade -y

sudo apt install -y bc bison build-essential ccache curl flex g++-multilib gcc-multilib \
  git git-lfs gnupg gperf imagemagick lib32readline-dev lib32z1-dev libelf-dev liblz4-tool \
  libncurses5-dev libncurses-dev libsdl1.2-dev libssl-dev libxml2 libxml2-utils lzop pngcrush \
  rsync schedtool squashfs-tools xsltproc zip unzip zlib1g-dev python3 python-is-python3 openjdk-17-jdk
```

### Install Google's `repo` tool
```bash
mkdir -p ~/.bin
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/.bin/repo
chmod a+x ~/.bin/repo

# Add to PATH if not already present
echo 'export PATH=~/.bin:$PATH' >> ~/.bashrc
source ~/.bashrc
```

### Configure Git & Git LFS
```bash
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
git lfs install
```

### Setup Ccache
Ccache caches compiled C/C++ objects, cutting future build times down from a couple of hours to 20-30 minutes.

```bash
echo 'export USE_CCACHE=1' >> ~/.bashrc
echo 'export CCACHE_EXEC=/usr/bin/ccache' >> ~/.bashrc
echo 'export CCACHE_DIR=$HOME/.ccache' >> ~/.bashrc
source ~/.bashrc

# Allocate 50-100 GB for cache
ccache -M 50G
ccache -o compression=true
```

---

## 2. Syncing Evolution X Source (Android 17 - cnb)

Create a workspace directory and initialize the manifest:

```bash
mkdir -p ~/evolution
cd ~/evolution

repo init -u https://github.com/Evolution-X/manifest -b cnb --git-lfs
```

Pull the source tree:
```bash
repo sync -c -j$(nproc --all) --force-sync --no-clone-bundle --no-tags
```

> **Note:** If your download gets interrupted or stalls due to rate limits, drop the thread count:
> ```bash
> repo sync -c -j4 --force-sync --no-clone-bundle --no-tags
> ```

---

## 3. Cloning Device Trees & HALs (lineage-24.0)

Source: [WibuRachSapphireTree/Script (branch 24.0)](https://github.com/WibuRachSapphireTree/Script/tree/24.0)

From the root of your source directory (`~/evolution`), run the following commands:

### Clone Device Trees
```bash
rm -rf device/xiaomi/sapphire-kernel
git clone --depth 1 -b lineage-24.0 https://github.com/WibuRachSapphireTree/device_xiaomi_sapphire-kernel device/xiaomi/sapphire-kernel

rm -rf device/xiaomi/sepolicy
git clone --depth 1 -b 17 https://github.com/WibuRachSapphireTree/device_xiaomi_sepolicy device/xiaomi/sepolicy

rm -rf device/xiaomi/sapphire
git clone --depth 1 -b lineage-24.0 https://github.com/WibuRachSapphireTree/device_xiaomi_sapphire device/xiaomi/sapphire

rm -rf vendor/xiaomi/sapphire
git clone --depth 1 -b lineage-24.0 https://github.com/WibuRachSapphireTree/vendor_xiaomi_sapphire vendor/xiaomi/sapphire

rm -rf hardware/xiaomi
git clone --depth 1 -b lineage-24.0 https://github.com/LineageOS/android_hardware_xiaomi hardware/xiaomi

rm -rf hardware/dolby
git clone --depth 1 https://github.com/WibuRachSapphireTree/hardware_dolby hardware/dolby

rm -rf device/xiaomi/miuicamera-sapphire
git clone --depth 1 -b lineage-24.0 https://github.com/WibuRachSapphireTree/device_xiaomi_miuicamera-sapphire device/xiaomi/miuicamera-sapphire

rm -rf vendor/xiaomi/miuicamera-sapphire
git clone --depth 1 -b lineage-24.0 https://github.com/WibuRachSapphireTree/vendor_xiaomi_miuicamera-sapphire vendor/xiaomi/miuicamera-sapphire
```

### Clone Qualcomm SM6225 HALs
```bash
rm -rf hardware/qcom-caf/common
git clone --depth 1 -b lineage-24.0 https://github.com/sapphire-sm6225/android_hardware_qcom-caf_common.git hardware/qcom-caf/common

rm -rf hardware/qcom-caf/sm6225/audio/agm
git clone --depth 1 -b lineage-24.0-caf-sm6225 https://github.com/sapphire-sm6225/vendor_qcom_opensource_agm.git hardware/qcom-caf/sm6225/audio/agm

rm -rf hardware/qcom-caf/sm6225/audio/pal
git clone --depth 1 -b lineage-24.0-caf-sm6225 https://github.com/sapphire-sm6225/vendor_qcom_opensource_arpal-lx.git hardware/qcom-caf/sm6225/audio/pal

rm -rf hardware/qcom-caf/sm6225/data-ipa-cfg-mgr
git clone --depth 1 -b lineage-24.0-caf-sm6225 https://github.com/sapphire-sm6225/vendor_qcom_opensource_data-ipa-cfg-mgr.git hardware/qcom-caf/sm6225/data-ipa-cfg-mgr

rm -rf hardware/qcom-caf/sm6225/dataipa
git clone --depth 1 -b lineage-24.0-caf-sm6225 https://github.com/sapphire-sm6225/vendor_qcom_opensource_dataipa.git hardware/qcom-caf/sm6225/dataipa

rm -rf hardware/qcom-caf/sm6225/display
git clone --depth 1 -b lineage-24.0-caf-sm6225 https://github.com/sapphire-sm6225/hardware_qcom_display.git hardware/qcom-caf/sm6225/display

rm -rf hardware/qcom-caf/sm6225/media
git clone --depth 1 -b lineage-24.0-caf-sm6225 https://github.com/sapphire-sm6225/hardware_qcom_media.git hardware/qcom-caf/sm6225/media

rm -rf hardware/qcom-caf/sm6225/audio/primary-hal
git clone --depth 1 -b lineage-24.0-caf-sm6225 https://github.com/sapphire-sm6225/hardware_qcom_audio.git hardware/qcom-caf/sm6225/audio/primary-hal

rm -rf device/qcom/sepolicy_vndr/sm6225
git clone --depth 1 -b lineage-24.0-caf-sm6225 https://github.com/sapphire-sm6225/device_qcom_sepolicy_vndr.git device/qcom/sepolicy_vndr/sm6225
```

---

## 4. Private Signing Keys (vendor/lineage-priv)

The device tree expects private certificates via:
```makefile
-include vendor/lineage-priv/keys/keys.mk
```
Using the keys package from [Huy7684/Keys-for-unofficial-rom](https://github.com/Huy7684/Keys-for-unofficial-rom) signs your build with consistent release keys instead of default AOSP test keys (helps avoid signature mismatches and CTS test-key flags).

Extract the zip inside `vendor/`:

```bash
cd ~/evolution/vendor
curl -sL -o Keys.zip https://raw.githubusercontent.com/Huy7684/Keys-for-unofficial-rom/main/Keys.zip
unzip -o Keys.zip
rm Keys.zip
cd ~/evolution
```

Verify that the makefile is present:
```bash
ls -l vendor/lineage-priv/keys/keys.mk
```

---

## 5. [Optional] NFC Patch (FixNFC)

If your device is the NFC variant (`sapphiren`) or you experience NFC initialization issues, apply the automated patch from [WibuRachSapphireTree/FixNFC](https://github.com/WibuRachSapphireTree/FixNFC):

```bash
cd ~
git clone https://github.com/WibuRachSapphireTree/FixNFC.git
cd FixNFC
chmod +x FixNFC.sh

# Preview changes without modifying anything
./FixNFC.sh --check ~/evolution

# Apply changes to your source tree (do NOT run with sudo)
./FixNFC.sh ~/evolution

cd ~/evolution
```

---

## 6. Building

Set up the environment:
```bash
. build/envsetup.sh
```

Lunch the target for `sapphire`:
```bash
# Debuggable build (recommended for testing):
lunch lineage_sapphire-cp2a-userdebug

# Or user build:
# lunch lineage_sapphire-cp2a-user
```

Start the build:
```bash
m evolution 2>&1 | tee build.log
```

Once completed, the flashable zip will be in:
```
out/target/product/sapphire/evolution_sapphire-*.zip
```

---

## Tips & Troubleshooting

### Out of Memory / Ninja killed
Android 17 builds can easily exceed 24-30 GB memory usage during link stages. If you see processes getting killed without an obvious error, set up a 32 GB swapfile:

```bash
sudo fallocate -l 32G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

Make it permanent by adding this line to `/etc/fstab`:
```
/swapfile none swap sw 0 0
```

### Cleaning between builds
* Changed trees or device flags:
  ```bash
  m installclean
  ```
* Switching branches or recovering from broken ninja cache:
  ```bash
  make clean
  ```

### Check Ccache stats
```bash
ccache -s
```
If `Cache hit rate` is growing, your setup is working as intended.
