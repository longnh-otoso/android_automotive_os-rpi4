# Android Automotive (AAOS 15) on Raspberry Pi 4

This repository contains instructions and scripts for building and installing Android Automotive on a Raspberry Pi 4.

## Prerequisites

Before we do this, we need the following environment:

- A 64-bit Linux build machine with more than 32GB RAM and at least 400GB free storage
- Ubuntu 22.04.3 

- A Raspberry Pi 4 (any model)
- Touchscreen display or monitor for the Raspberry Pi
- At least a 16GB SD card for flashing the image
- SD card speed should be at least 12.5 MB/s to avoid performance issues
- Basic knowledge of using the terminal

## Step 1: Set Up Your Build Environment

1. **Install necessary packages:**
   ```bash
   sudo apt update 
   
   sudo apt-get install openjdk-11-jdk git python3 bison flex git  bc build-essential curl g++-multilib gcc-multilib  lib32ncurses5-dev lib32readline-dev liblz4-tool libncurses5 libncurses5-dev meson cmake pkg-config

   sudo apt install python-is-python3

   sudo apt install pip
   
   sudo apt install repo
   
   sudo apt install unzip 
   
   sudo apt-get install -y mtools dosfstools
   ```

## Step 2: Download and Build the Android Source

1. **Create a directory for the Android source:**
   ```bash
   mkdir ~/android
   cd ~/android
   ```

2. **Initialize and sync the Android repository:**
   ```bash
   repo init -u https://android.googlesource.com/platform/manifest --depth=1 -b android-14.0.0_r37
   git clone https://github.com/android-rpi/local_manifests .repo/local_manifests -b arpi-14
   repo sync -j$(nproc)
   ```

3. **Set up the build environment:**
   ```bash
   source build/envsetup.sh
   lunch rpi4-eng
   ```

4. **Build the Android system images:**
   ```bash
   make ramdisk systemimage vendorimage -j$(nproc)
   ```

## Step 3: Download and Build the Kernel

1. **Create a directory for the kernel source:**
   ```bash
   mkdir ~/kernel
   cd ~/kernel
   ```

2. **Initialize and sync the kernel repository:**
   ```bash
   repo init -u https://github.com/android-rpi/kernel_manifest -b arpi14-6.1.62
   repo sync -j$(nproc)
   ```

3. **Build the kernel:**
   ```bash
   build/build.sh
   ```

## Step 4: Prepare the SD Card

1. **Partition the SD card:**
   Use `fdisk` to create the following partitions:
   - **p1:** 128MB, W95 FAT32 (LBA), Bootable
   - **p2:** 1152MB, primary
   - **p3:** 128MB, primary
   - **p4:** remaining space, primary, ext4

   ```bash
   sudo fdisk /dev/sdX
   # Follow fdisk commands to create the partitions as described above.
   ```

2. **Format the partitions:**
   ```bash
   sudo mkfs.vfat /dev/sdX1
   sudo mkfs.ext4 /dev/sdX2
   sudo mkfs.ext4 /dev/sdX3
   sudo mkfs.ext4 -L userdata /dev/sdX4
   ```

## Step 5: Flash the Pre-built Images to the SD Card

1. **Write the Android system images to the respective partitions:**
   ```bash
   cd ~/android/out/target/product/rpi4
   sudo dd if=system.img of=/dev/sdX2 bs=1M
   sudo dd if=vendor.img of=/dev/sdX3 bs=1M
   ```

2. **Copy the firmware and ramdisk to the boot partition:**
   ```bash
   sudo mount /dev/sdX1 /mnt
   sudo cp ~/android/device/arpi/rpi4/boot/* /mnt/
   sudo cp ramdisk.img /mnt/
   sudo umount /mnt
   ```

3. **Copy the kernel binaries to the boot partition:**
   ```bash
   sudo mount /dev/sdX1 /mnt
   sudo cp ~/kernel/out/arpi14-6.1/dist/Image.gz /mnt/
   sudo cp ~/kernel/out/arpi14-6.1/dist/bcm2711-rpi-*.dtb /mnt/
   sudo cp ~/kernel/out/arpi14-6.1/dist/vc4-kms-v3d-pi4.dtbo /mnt/overlays/
   sudo umount /mnt
   ```

## Final Steps

1. **Insert the SD card into your Raspberry Pi 4 and power it on.**
2. **The Raspberry Pi should boot into Android Automotive.**

## References

- [Downloading the Android source](http://source.android.com/source/downloading.html)
- [Building Android](http://source.android.com/source/building.html)
- [Kernel source for Raspberry Pi](https://github.com/android-rpi/kernel_manifest/tree/arpi14-6.1.62)
- [Building kernels](https://source.android.com/setup/build/building-kernels)
- [Device manifest for Raspberry Pi 4](https://github.com/android-rpi/device_arpi_rpi4)
