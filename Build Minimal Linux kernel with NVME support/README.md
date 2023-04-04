
# How to Build Minimal Linux OS kernel Using Busybox and Qemu on ARM with NVME support

Here I have Build kernel of Linux 4.19.18 using Busybox and Qemu.




## System Requirements

- Linux 4.19.18
- Busybox 1.35.0
- Qemu
- OS- ARM64
- Compiler: aarch64-linux-gnu



## Install Dependencies

```bash 
sudo apt install qemu-system-arm qemu-system-aarch64 qemu-utils gcc-aarch64-linux-gnu build-essential lzop bison libncurses-dev git make gcc fakeroot ncurses-dev xz-utils libssl-dev bc flex libelf-dev

```
##### Create Project Directory

```bash 
mkdir qemu
cd qemu
```
## Build Linux 4.19.18
1. Download the kernel
```bash 
wget https://cdn.kernel.org/pub/linux/kernel/v4.x/linux-4.19.18.tar.gz
```
2. Extract the kernel & change Directory
```bash 
tar -xvf linux-4.19.18.tar.gz

cd linux-4.19.18
```
3. MAKE default config
```bash 
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- defconfig
```
4. Open Config & Enable NVME, PCIe Support
```bash 
vim .config
```
5. Change the following in config files
- CONFIG_CONFIGFS_FS=y

- CONFIG_NVME_CORE=y

- CONFIG_BLK_DEV_NVME=y

- CONFIG_NVME_TARGET=y

- CONFIG_RTC_NVMEM=y

- CONFIG_NVMEM=y

- CONFIG_KGDB =y
6. MAKE .config compatible 
```bash 
yes "" | make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- oldconfig -j$(nproc)
```
7. Compile the kernel
```bash 
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- -j$(nproc)
```
8. Open scripts/dtc/dtc-lexer.lex.c and remove the following
```bash 
vim scripts/dtc/dtc-lexer.lex.c
```
Search yyloc and remove the line
```bash 
YYLTYPE yylloc;
```
9. Compile the kernel again
```bash 
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- -j$(nproc)
```
:star2::star2::star2:

**_Congratulations!! Kernel is compiled successfully and located at arch/arm64/boot directory_**

10. Check the boot image
```
ls -lah arch/arm64/boot/Image
```
Back to parent directory

```
cd ../
```
## Build Busybox
 1. Download Busybox 1.35.0
 ```
wget https://busybox.net/downloads/busybox-1.35.0.tar.bz2
```
2. Extract busybox
```
tar -xvf busybox-1.35.0.tar.bz2
```
3. Change directory & Modify Build Config
```
cd busybox-1.35.0
```
Enable Settings > Build Options > Build static binary (no shared libs)
```
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- menuconfig -j$(nproc)
```
4. Compile Busybox
```
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- -j$(nproc)
```
## Create rootfs
1. Make directory for rootfs, copy files & change directory
```
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- install CONFIG_PREFIX=rootfs -j$(nproc)

cd rootfs
```
2. create required directories in rootfs
```
mkdir -pv {bin,sbin,etc/init.d,proc,sys,usr/{bin,sbin}}
```
3. Create etc/init.d/rcS file
```
vim etc/init.d/rcS
```
Add following lines
```
#!/bin/sh

#This is the first script called by init process

/bin/mount -a

echo /sbin/mdev>/proc/sys/kernel/hotplug

mdev -s
```
4. Make etc/init.d/rcS exicutable
```
chmod a+x etc/init.d/rcS
```
5. Create etc/fstab file
```
vim etc/fstab
```
Add following lines
```
#device mount-point type options dump fsck order

proc /proc proc defaults 0 0

tmpfs /tmp tmpfs defaults 0 0

sysfs /sys sysfs defaults 0 0

tmpfs /dev tmpfs defaults 0 0
```
6. Build rootfs.img image file
```
find . -print0 | cpio --null -ov --format=newc | gzip -9 > ../../rootfs.img
```
:star2::star2::star2:

**_Congratulations!! rootfs Image is created successfully_**

7. Back to the parent directory
```
cd ../../
```
### Create Virtual NVME Drive
Create nvme.img of 4GB
```
dd if=/dev/zero of=nvme.img bs=1M count=4096
```
### Build System using QEMU
```
qemu-system-aarch64 -M virt -cpu cortex-a53 -smp 2 -m 4096M -kernel linux-4.19.18/arch/arm64/boot/Image -nographic -append "console=ttyAMA0 rdinit=linuxrc" -initrd rootfs.img
```
** Close the Qemu Environment **
```
ctrl + A + X

```
### Build the system with NVME
```
qemu-system-aarch64 -M virt -cpu cortex-a53 -smp 2 -m 4096M -kernel linux-4.19.18/arch/arm64/boot/Image -nographic -append "console=ttyAMA0 rdinit=linuxrc" -initrd rootfs.img -drive file=nvme.img,format=raw,if=none,id=nvme -device nvme,serial=deadbeef,drive=nvme
```

:star2::star2::star2:

ENJOY!!!

:star2::star2::star2:
## Authors

- [@pkchowdhury](https://www.github.com/pkchowdhury)
- [@flyluman](https://www.github.com/flyluman)
