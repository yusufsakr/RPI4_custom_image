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
     >> (a) boot : [100 - 500] MB = FAT  format 
     | (b) root : Remaining size = Ext4 format 

<div>
<img src="https://github.com/yusufsakr/RPI4_build_root/assets/91801319/b474bb4d-7a52-4cc7-b34d-0069cee86649" width="300">
<img src="https://github.com/yusufsakr/RPI4_build_root/assets/91801319/70804479-b0e2-4667-ab82-6b3b9816f9fb" width="300">
<img src="https://github.com/yusufsakr/RPI4_build_root/assets/91801319/72df9264-0c8b-4392-b42e-fe20597211ed" width="300">
<div>


```
helloo
```
