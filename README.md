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
