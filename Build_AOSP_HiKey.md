#Build Android Open Source Project (AOSP) for 96Boards HiKey

##Install 64bit Ubuntu 14.04 / 16.04

###Hardware Requirement
* 200GB of free disk space

###Update System
```
sudo apt-get update
sudo apt-get upgrade
sudo apt-get dis-upgrade
```

###Set User Privilege
```
sudo visudo
```
Add Lines (replacing USER with your username):
```
USER    ALL= NOPASSWD: /bin/mount
USER    ALL= NOPASSWD: /bin/umount
USER    ALL= NOPASSWD: /sbin/mkfs.fat
USER    ALL= NOPASSWD: /bin/cp
```

##Install Required Packages

##Install JDK
```
sudo apt-get install openjdk-8-jdk
sudo update-alternatives --config java
sudo update-alternatives --config javac
```

##Install Packages
```
sudo apt-get install git-core gnupg flex bison gperf build-essential zip curl zlib1g-dev gcc-multilib g++-multilib libc6-dev-i386 lib32ncurses5-dev x11proto-core-dev libx11-dev lib32z-dev ccache libgl1-mesa-dev libxml2-utils xsltproc unzip mtools android-tools-adb android-tools-fastboot python-serial
```

##Configure Environment
###Configuring USB Access
```
wget -S -O - http://source.android.com/source/51-android.rules | sed "s/<username>/$USER/" | sudo tee >/dev/null /etc/udev/rules.d/51-android.rules
```
* Add VID=18d1:PID=4ee7 to 51-android.rules
```
sudo udevadm control --reload-rules
```

###Installing Repo
```
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/repo
chmod a+x ~/repo
sudo mv repo /usr/sbin/
```

###Configure Git
```
git config --global user.name "Your Name“
git config --global user.email you@example.com
```

##Download Android Source & Binaries
###Android Source Tree
```
mkdir ~/android && cd $_
repo init -u https://android.googlesource.com/platform/manifest -b master
repo sync -j4
```

###HDMI Binaries
```
wget https://dl.google.com/dl/android/aosp/linaro-hikey-20160226-67c37b1a.tgz
tar xzf linaro-hikey-20160226-67c37b1a.tgz
./extract-linaro-hikey.sh
```

##Build System Image
###Clean Build
```
make clobber
```

###Build Script
```
export _JAVA_OPTIONS="-Xmx4096M"
source build/envsetup.sh
lunch hikey-userdebug
make -j4
```

###Output Path
* out/target/product/hikey

##Download Kernel Source
###Download Kernel Source
```
cd ~
git clone https://android.googlesource.com/kernel/hikey-linaro
cd hikey-linaro
git checkout -b android-hikey-linaro-4.4 origin/android-hikey-linaro-4.4
```

###Compile Kernel
```
make ARCH=arm64 hikey_defconfig
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-android- -j4
```

###Output Path
* arch/arm64/boot/
```
cp arch/arm64/boot/dts/hisilicon/hi6220-hikey.dtb ~/android/device/linaro/hikey-kernel/
cp arch/arm64/boot/Image-dtb ~/android/device/linaro/hikey-kernel/
```

##Build Kernel Image
###Build Kernel
```
cd ~/android
source build/envsetup.sh
lunch hikey-userdebug
make bootimage -j2
```

###Output Path
* out/target/product/hikey/boot.img
* out/target/product/hikey/ramdisk.img

##Flash Image
###Enter Recovery Mode
* Make sure pin1-pin2 and pin3-pin4 on J15 are linked

###Flash Bootloader
* ex: /dev/ttyUSB1
```
sudo ./device/linaro/hikey/installer/flash-all.sh /dev/ttyUSB1
```

###Enter Fastboot Mode
* Make sure pin1-pin2 and pin5-pin6 on J15 are linked

###Flash System
```
sudo fastboot flash boot out/target/product/hikey/boot.img
sudo fastboot flash -w system out/target/product/hikey/system.img
```

###Enter ADB Mode
* Make sure pin3-pin4 and pin5-pin6 on J15 are unlinked

##References
* AOSP - Downloading and Building [http://source.android.com/source/devices.html](http://source.android.com/source/devices.html)
* CE AOSP RPB HiKey 15.12 Build [https://github.com/96boards/documentation/wiki/CE-AOSP-RPB-HiKey-15.12-Build](https://github.com/96boards/documentation/wiki/CE-AOSP-RPB-HiKey-15.12-Build)
* CE AOSP RPB HiKey 15.12 Install [https://github.com/96boards/documentation/wiki/CE-AOSP-RPB-HiKey-15.12-Install](https://github.com/96boards/documentation/wiki/CE-AOSP-RPB-HiKey-15.12-Install)
* Reference Bootloader Hikey [https://github.com/96boards/documentation/wiki/Reference-Bootloader-Hikey](https://github.com/96boards/documentation/wiki/Reference-Bootloader-Hikey)
* 编译HiKey内核 [http://www.voidcn.com/blog/pcsxk/article/p-6168233.html](http://www.voidcn.com/blog/pcsxk/article/p-6168233.html)
