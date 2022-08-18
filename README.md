# Compiling and Deploying BeagleBone Black Kernel

## - Prerequisites

```bash
sudo apt update
sudo apt upgrade
```

Gparted :

```bash
sudo apt-get install gparted
```

Minicom :

```bash
sudo apt-get install minicom
```

MkImage :

```bash
sudo apt-get install uboot-mkimage
```



## - Installing ARM cross compiler toolchain

Install:

```bash
sudo apt install git bc bison flex libssl-dev make libc6-dev libncurses5-dev
sudo apt install crossbuild-essential-armhf
```

To check if it is correctly installed and the version:

```bash
arm-linux-gnueabihf-gcc --version	
```



## - U-boot

U-boot is an open source universal bootloader for Linux systems. It  supports features like TFTP, DHCP, NFS etc… In order to boot the kernel, a valid kernel image (uImage) is required.

It is not possible to  explain u-boot here, as it is beyond the scope of this post. So, we will see how to produce a bootable image using U-boot. You can either follow the below steps to compile your own U-boot image or you could just get a prebuilt image.

Download:

```bash
mkdir -p ~/Beaglebone
cd ~/Beaglebone
git clone -b v2022.04 https://github.com/u-boot/u-boot --depth=1
```

Compile :

```bash
cd u-boot
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- distclean
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- am335x_evm_defconfig
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf-
```

## - Building a kernel for beaglebone black

### Cloning the Kernel

```bash
cd ~/Beaglebone
git clone https://github.com/beagleboard/linux.git
```

Go to the linux directory and ensure that you have cloned the correct repo by executing the following command :

```bash
cd linux/
git remote -v 
```

- You should get this below output

  ```bash
  origin https://github.com/beagleboard/linux.git (fetch)
  origin https://github.com/beagleboard/linux.git (push)
  ```

Default Configuration ;

```bash
sudo make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- omap2plus_defconfig
```

Compile the kernel :

```bash
sudo make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- uImage dtbs LOADADDR=0x80008000 -j8
```

## - Install Kernel Modules

```bash
sudo make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- -j8 modules
```



## - Formatting the SD card

```bash
sudo gparted
```

- Insert your sd card by means of card reader and open Gparted. Select  your sd card from the top right corner. it will be something like this,  “/dev/sdb”

- Right click on the rectangular area showing your sd card name and select unmount as we need to unmount the existing partitions. 
- Then, delete the existing partitions by again right clicking and selecting “delete”. This will delete all your files in sd card.
- We need two partitions in order to boot the kernel, one is for storing  the bootable images and another one is for storing the minimal RFS(Root  File System).

1. Click Add button. Then, create partition for storing ***BOOT*** by entering the options below

```bash
    New size: 1024 MB
    File System: FAT32
    Label: BOOT
```

2. Click Add button. Then, create another partition for storing ***rootfs*** by entering the options below

```bash
    New Size: (Remaining)
    File System: EXT4
    Label: rootfs
```

- Finally, click the green tick mark at the menu bar. 

## -  Install Kernel Modules on SD Card

```bash
sudo make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- INSTALL_MOD_PATH=/media/$USER/rootfs/ modules_install
```

## - Root File System (RFS)

Clone the Root file system :

```bash
cd ~/Beaglebone
git clone https://github.com/Saishivareddy/Rootfs/
cd Rootfs/
```

Copy the files into the partition in SD Card ;

```bash
sudo tar -xvf rootfs.tar.xz -C /media/$USER/rootfs/
cd /media/$USER/rootfs/rootfs/
sudo mv ./* ../
cd ../
sudo rmdir rootfs
```

## - Copy Image and dtbs on SD Card(Boot)

```bash
cd ~\Beaglebone/linux/
cp arch/arm/boot/uImage /media/$USER/BOOT
cp arch/arm/boot/dts/am335x-boneblack.dtb /media/$USER/BOOT
```

```bash
vim /media/$USER/BOOT/uEnv.txt
```

- Copy the below code into that

```bash
console=ttyS0,115200n8
netargs=setenv bootargs console=ttyO0,115200n8 root=/dev/mmcblk0p2 ro rootfstype=ext4 rootwait debug earlyprintk mem=512M
netboot=echo Booting from microSD ...; setenv autoload no ; load mmc 0:1 ${loadaddr} uImage ; load mmc 0:1 ${fdtaddr} am335x-boneblack.dtb ; run netargs ; bootm ${loadaddr} - ${fdtaddr}
uenvcmd=run netboot
```

- After completing the above steps you can find the following files in BOOT partition of sd card.
  - uImage
  - am335x-boneblack.dtb
  - uEnv.txt
