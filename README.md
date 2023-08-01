# RPI4_build_root
Embedded Linux Build Root for Raspberry Pi 4 Step-By-Step.

## Overview
In this tutoeial we will build and configure a minimal image for raspberry pi 4 with customized toolchain, uboot, kernel and root filesystem.

## Required hardware
1) Linux Machine OS (ex: Ubuntu 20.04 LTS)
2) Raspberry Pi 4
3) SD Card >= 2gb
4) SD Card Reader

## Major Steps
1) Configure the SD Card
2) Configure and build the Toolchain
3) Building the Bootloader
4) Config and Build the Kernel
5) Create the root filesystem
6) Boot the RPI4

## SD Card Configuration
1. Plug in the SD Card
2. Open Disks app (Ubuntu)
3. Format the SD Card
4. Creat two partitions
     * (a) boot : [100 - 500] MB = FAT  format 
     * (b) root : Remaining size = Ext4 format
5. Mount the Partitions from the "Play" Symbol

<div>
<img src="https://github.com/yusufsakr/RPI4_build_root/assets/91801319/b474bb4d-7a52-4cc7-b34d-0069cee86649" width="300">
<img src="https://github.com/yusufsakr/RPI4_build_root/assets/91801319/70804479-b0e2-4667-ab82-6b3b9816f9fb" width="300">
<img src="https://github.com/yusufsakr/RPI4_build_root/assets/91801319/72df9264-0c8b-4392-b42e-fe20597211ed" width="300">
<div>

## Install some dependecies
```
sudo apt -y install gcc-arm-linux-gnueabihf binutils-arm-linux-gnueabihf
sudo apt-get -y install bison flex bc libssl-dev make gcc automake chrpath g++ git gperf gawk help2man libexpat1-dev libncurses5-dev libsdl1.2-dev libtool libtool-bin libtool-doc python2.7-dev texinfo
```
     
## Toolchain
The toolchain is an essential tool to compile the source code to executable one runnning on the RPI4.
We will use Crosstool-NG.
[Crosstool-NG build Explanation](https://crosstool-ng.github.io/docs/toolchain-construction/)

### Download crosstool-NG source
```
cd ~
git clone https://github.com/crosstool-ng/crosstool-ng
cd crosstool-ng/

# Switch to the latest release
git checkout crosstool-ng-1.24.0 -b 1.24.0
```
### Build and Install crosstool-NG
```
./bootstrap
./configure --prefix=${PWD}
make
make install
export PATH="${PWD}/bin:${PATH}"
```
### Creating Toolchain for RPI4
1) Frst, we must configure the crosstool-NG to build the toolchain for the RPI4.
You can find a sample toolchain for RPI3 ...
```
ct-ng list-samples
```
2) At the time of writing, there are no configurations for RPi_4. 
To avoid creating one from scratch, we can use an existing configuration of RPi_3 and modify it to match RPi_4
```
ct-ng show-aarch64-rpi3-linux-gnu
```
3) Select aarch64-rpi3-linux-gnu as a base-line configuration
```
ct-ng aarch64-rpi3-linux-gnu
```
4) To configure the base line of the toolchain, we first need to know the CPU model of the RPI_4 which is
   > Broadcom BCM2711, Quad core Cortex-A72 (ARM v8) 64-bit SoC @ 1.5GHz
```
ct-ng menuconfig
```
<img src="https://github.com/yusufsakr/RPI4_build_root/assets/91801319/8b22e03e-f24b-4de5-bc3d-069019b0afed" width="500">

5) We will make 3 changes :
   * Allow Extending the toolchain after it is created ...
     > Paths and misc options -> Render the toolchain read-only = false
   *  Change the ARM Cortex core ...
      > Target options -> Emit assembly for CPU: Change cortex-a53 to cortex-a72
   * Chane the tuple’s vendor string ...
     > Toolchain options -> Tuple’s vendor string: Change rpi3 torpi4

6) Build the Toolchain :
```
ct-ng build
```
7) The toolchain will be named aarch64-rpi4-linux-gnu and created in ...
   > ${HOME}/x-tools/aarch64-rpi4-linux-gnu

## Bootloader
The bootloader’s job is to set up the system to a basic level and load the kernel.

### Download u-boot source
```
cd ~
git clone git://git.denx.de/u-boot.git
cd u-boot
git checkout v2021.10 -b v2021.10
```

### Configure U-Boot
```
export PATH=${HOME}/x-tools/aarch64-rpi4-linux-gnu/bin/:$PATH
export CROSS_COMPILE=aarch64-rpi4-linux-gnu-
make rpi_4_defconfig
```

### Build u-boot
```
make
```
### Install U-Boot
1. We will copy the u-boot bary file to the "boot" partition of the SD Card we create earlier.
It could be in another destination.
```
sudo cp u-boot.bin /mnt/boot
```

**NOTE:**
Raspberry Pi has its own proprietary bootloader, which is loaded by the ROM code and is capable of loading the kernel. However, since I’d like to use the open source u-boot, I’ll need to configure the Raspberry Pi boot loader to load u-boot and then let u-boot load the kernel.

2. Download Raspberry Pi firmware/boot directory
```
cd ~
svn checkout https://github.com/raspberrypi/firmware/trunk/boot
```

3. Copy RPI4 bootloader into boot partition
```
sudo cp boot/{bootcode.bin,start4.elf} /mnt/boot/
```

4. Let RPI4 bootloader load U-Boot
```
cat << EOF > config.txt
enable_uart=1
arm_64bit=1
kernel=u-boot.bin
EOF

sudo mv config.txt /mnt/boot/
```

## Kernel
Now we will compile the kernel Image.

### Download the Kernel Source
Despite that the original Linux kernel from [Linus Torvalds](https://github.com/torvalds/linux) should work. But, using the RPI's fork of it would be more stable.
```
cd ~
git clone --depth=1 -b rpi-5.10.y https://github.com/raspberrypi/linux.git
cd linux
```

### Config and Build the Kernel
We just use the default config for the RPI4 model B, by the knowledge of its [Specs](https://www.raspberrypi.com/products/raspberry-pi-4-model-b/specifications/).
```
PATH=$PATH:~/x-tools/aarch64-rpi4-linux-gnu/bin
make ARCH=arm64 CROSS_COMPILE=aarch64-rpi4-linux-gnu- bcm2711_defconfig
make -j$(nproc) ARCH=arm64 CROSS_COMPILE=aarch64-rpi4-linux-gnu-
```

### Install the kernel and device tree
Now we copy the kernel image and device tree binary (*.dtb) into the boot partition on the SD card.
```
sudo cp arch/arm64/boot/Image /mnt/boot
sudo cp arch/arm64/boot/dts/broadcom/bcm2711-rpi-4-b.dtb /mnt/boot/
```

## Root filesystem
We will Customize the Root filesystem folders and files.

### Create Directories
```
mkdir rootfs
cd rootfs
mkdir {bin,dev,etc,home,lib64,proc,sbin,sys,tmp,usr,var}
mkdir usr/{bin,lib,sbin}
mkdir var/log

# Create a symbolink lib pointing to lib64
ln -s lib64 lib

tree -d
# Output ...
# .
# ├── bin
# ├── dev
# ├── etc
# ├── home
# ├── lib -> lib64
# ├── lib64
# ├── proc
# ├── sbin
# ├── sys
# ├── tmp
# ├── usr
# │   ├── bin
# │   ├── lib
# │   └── sbin
# └── var
#     └── log

# 16 directories

# Change the owner of the directories to be root
# Because current user doesn't exist on target device
sudo chown -R root:root *
```

### Build and Install Busybox
We’ll use Busybox for essential Linux utilities such as shell. So, we need to install it to the rootfs directory just created.

1) Download Source code
```
wget https://busybox.net/downloads/busybox-1.33.2.tar.bz2
tar xf busybox-1.33.2.tar.bz2
cd busybox-1.33.2/
```
2) Configuring
```
CROSS_COMPILE=${HOME}/x-tools/aarch64-rpi4-linux-gnu/bin/aarch64-rpi4-linux-gnu-
make CROSS_COMPILE="$CROSS_COMPILE" defconfig
Change the install directory to be the one just created
sed -i 's%^CONFIG_PREFIX=.*$%CONFIG_PREFIX="/home/hechaol/rootfs"%' .config
```
3) Building
```
make CROSS_COMPILE="$CROSS_COMPILE"
```

4) Installing
```
Use sudo because the directory is now owned by root
sudo make CROSS_COMPILE="$CROSS_COMPILE" install
```
### Install required libraries
* Next, we install some shared libraries required by Busybox. We can find those libraries by the following command ...
```
readelf -a ~/rootfs/bin/busybox | grep -E "(program interpreter)|(Shared library)"
# Output ...
#       [Requesting program interpreter: /lib/ld-linux-aarch64.so.1]
#  0x0000000000000001 (NEEDED)             Shared library: [libm.so.6]
#  0x0000000000000001 (NEEDED)             Shared library: [libresolv.so.2]
#  0x0000000000000001 (NEEDED)             Shared library: [libc.so.6]
```
* Coping these files from the toolchain’s sysroot directory to the rootfs/lib directory ...
```
export SYSROOT=$(aarch64-rpi4-linux-gnu-gcc -print-sysroot)
sudo cp -L ${SYSROOT}/lib64/{ld-linux-aarch64.so.1,libm.so.6,libresolv.so.2,libc.so.6} ~/rootfs/lib64/
```

### Create device nodes
Two device nodes are needed by Busybox.
```
cd ~/rootfs
sudo mknod -m 666 dev/null c 1 3
sudo mknod -m 600 dev/console c 5 1
```

### Copy the rootfs contents to the SD Card rot partition
```
sudo cp ~/rootfs/ /mnt/root/
```

## Boot the Board
Finally, with the Puzzle pieces are ready, we can boot the board using the U-Bootwe built earlier.

### Configure U-Boot
We need to configure the u-boot so that it can pass the correct kernel commandline and device tree binary to kernel. For simplicity, I’ll use the Busybox shell as the init program. 

1) Creating txt file for the boot command ...
   * Load the kernel image from partition 1 (boot partition) into memory.
   * Load the initramfs from partition 1 (boot partition) into memory.
   * Set kernel commandline.
   * Boot using the given kernel, device tree binary and initramfs.
```
cd ~/u-boot/
cat << EOF > boot_cmd.txt
fatload mmc 0:1 \${kernel_addr_r} Image
setenv bootargs "console=serial0,115200 console=tty1 root=/dev/mmcblk0p2 rw rootwait init=/bin/sh"
booti \${kernel_addr_r} - \${fdt_addr}
EOF
```

2) Generating the boot.scr file
```
~/u-boot/tools/mkimage -A arm64 -O linux -T script -C none -d boot_cmd.txt boot.scr
```

3) Copy the compiled boot script to boot partition
```
sudo cp boot.scr /mnt/boot/
```

### BOOTING IT :)
At Last we can boot the Image on RPI4.

1) Check on the boot partition files ...
```
tree /mnt/boot/
# OUTPUT ...
# /mnt/boot/
# ├── bcm2711-rpi-4-b.dtb
# ├── bootcode.bin
# ├── boot.scr
# ├── config.txt
# ├── Image
# ├── start4.elf
# ├── uRamdisk
# └── u-boot.bin

# 0 directories, 7 files
```
2) Unmount the SD Card
3) Plug the SD Card in the RPI4
4) Connect it with a Monitor (I didn't try the UART connection with the image yet).
5) Power it up
6) Now you should see the Busybox Shell if the image have been created successfully.

# Resources
* [Creating the RPI4 Toolchain](https://ilyas-hamadouche.medium.com/creating-a-cross-platform-toolchain-for-raspberry-pi-4-5c626d908b9d)
* [Hechao's Blog](https://hechao.li/2021/12/20/Boot-Raspberry-Pi-4-Using-uboot-and-Initramfs/)
* [link](https://forums.raspberrypi.com/viewtopic.php?f=98&t=314845)
* [Mastering Embedded Linux Programmin - Third Edition](https://www.amazon.com/Mastering-Embedded-Linux-Programming-potential/dp/1789530385)
* [linux From Scratch](https://www.linuxfromscratch.org/)
* [How Toolchan is Constructed](https://crosstool-ng.github.io/docs/toolchain-construction/)


Best Wishes :)
