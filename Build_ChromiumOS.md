#Build Chromium OS

##Install 64bit Ubuntu 14.04/16.04
###Hardware Requirement
* 200GB of free disk space

###Update System
```
sudo apt-get update
sudo apt-get upgrade
sudo apt-get dis-upgrade
```

###Making sudo a little more permissive
```
cd /tmp
cat > ./sudo_editor <<EOF
#!/bin/sh
echo Defaults \!tty_tickets > \$1          # Entering your password in one shell affects all shells 
echo Defaults timestamp_timeout=180 >> \$1 # Time between re-requesting your password, in minutes
EOF
chmod +x ./sudo_editor 
sudo EDITOR=./sudo_editor visudo -f /etc/sudoers.d/relax_requirements
```

##Install Required Packages
###Install Packages
```
sudo apt-get install git-core gitk git-gui subversion curl
```

###Install depot_tools
```
cd ~
git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
```
* Add Line to ~/.bashrc
```
export PATH=“$HOME”/depot_tools:"$PATH"
```

##Configure Environment
###Configure Git
```
git config --global user.name "Your Name“
git config --global user.email "you@example.com"
```

###Gerrit credentials setup
* Login https://chromium-review.googlesource.com/new-password
* Select “only chromium.googlesource.com”
* Copying this script and pasting it into a shell

##Download Chromium OS Source
###Chromium OS Source Tree
```
mkdir ~/chromiumos && cd $_
repo init -u https://chromium.googlesource.com/chromiumos/manifest.git --repo-url https://chromium.googlesource.com/external/repo.git
repo sync -j4
```

###SDK & chroot
```
cros_sdk
```

##Setup Build Target (First Time Only)
###Select Board
```
export BOARD=auron_yuna
./setup_board --board=${BOARD}
```
* add "--force" for clobber previously set up

###Set Kernel Source Tree
```
emerge-$BOARD -pv virtual/linux-sources
cros_workon --board=${BOARD} start sys-kernel/chromeos-kernel-3_XX
```

###Set the chronos user password
```
./set_shared_user_password.sh
```

###Accepting Additional Licenses
* edit /etc/make.conf.user file in your chroot
```
ACCEPT_LICENSE="*"
```

##Build System Image
###Build Package
```
./build_packages --board=${BOARD}
```

###Build Image
```
./build_image --board=${BOARD} --noenable_rootfs_verification test
```

###Output Path
* ~/trunk/src/build/images/${BOARD}/latest

##Flash Device
###Over 4GB USB Disk
```
cros flash usb:// ${BOARD}/latest
```

###Boot From USB
```
CTRL + U in booting
CTRL + ALT + F2 to Virtual Terminal 2
```

###Install to Disk
```
chromeos-install
reboot
```
* plug-out USB

* CTRL + ALT + F2 to Virtual Terminal 2
```
enable_dev_usb_boot
/usr/share/vboot/bin/make_dev_ssd.sh --remove_rootfs_verification --partitions 2
```

##Other then Ubuntu 14.04 Distribution
###Build failure in update_bootloaders.sh
#### Edit ~/update_bootloaders.sh
* Line 143 origin:
```
ESP_FS_DIR=$(mktemp -d /tmp/esp.XXXXXX)
```
* change to:
```
ESP_FS_DIR=$(mktemp -d /tmp/esp.XXXXXX)
ESP_LOOP_COPY=$(mktemp /tmp/esp-loop-copy.XXXXXX)
```

* Line 157 origin:
```
rm -rf "${ESP_FS_DIR}"
```
* change to:
```
rm -rf "${ESP_FS_DIR}"
rm -f "${ESP_LOOP_COPY}"
```

* Line 196 orgign:
```
sudo syslinux -d /syslinux "${ESP_DEV}"
```
* change to:
```
# We cannot syslinux ESP_DEV directly because of a race condition with udev,
# see http://crbug.com/508713
sudo dd if="${ESP_DEV}" of="${ESP_LOOP_COPY}" status=none
sudo syslinux -d /syslinux "${ESP_LOOP_COPY}"
sudo flock --wait 10 -x "${ESP_DEV}" \
dd if="${ESP_LOOP_COPY}" of="${ESP_DEV}" status=none
```

##References
* Chromium OS Developer Guide [https://www.chromium.org/chromium-os/developer-guide](https://www.chromium.org/chromium-os/developer-guide)
* Developer Information for Chrome OS Devices [https://www.chromium.org/chromium-os/developer-information-for-chrome-os-devices](https://www.chromium.org/chromium-os/developer-information-for-chrome-os-devices)
* Kernel Configuration [https://www.chromium.org/chromium-os/how-tos-and-troubleshooting/kernel-configuration](https://www.chromium.org/chromium-os/how-tos-and-troubleshooting/kernel-configuration)
* Licensing Handling For OS Builders [http://dev.chromium.org/chromium-os/licensing/building-a-distro](http://dev.chromium.org/chromium-os/licensing/building-a-distro)
* Build failure in update_bootloaders.sh [https://bugs.chromium.org/p/chromium/issues/detail?id=508713](https://bugs.chromium.org/p/chromium/issues/detail?id=508713)
