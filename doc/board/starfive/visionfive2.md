StarFive VisionFive2
====================

Before performing the steps described here, you must complete tasks on the following page.
https://wiki.gentoo.org/wiki/StarFive_VisionFive_2

Task Prerequisites

Building a system image
-----------------------
There are two philosophies when it comes to installing an operating system onto a SBC / embedded device such as the VisionFive 2.

The first involves writing a static system image, typically a squashfs, onto some form of media (typically eMMC, though NVMe is on the rise). The initramfs is able to load this image into RAM and use it as a rootfs; when an update is required the whole image is replaced as a single operation. There are advantages to this approach, particularly for embedded devices where users are not expected to update individual packages and recovery 'in-the-field' may be impractical, or to provide an A/B partition layout for updates. Systems configured in this way are also resilient when it comes to unexpected shutdowns as the only time that the rootfs storage volume is performing writes is when this image is being updated. This approach will be described as an 'embedded' installation going forward and called out where possible.

The second approach involves writing a rootfs onto some accessible storage media the device and using a package manager to install and update packages as required. For a Gentoo system this approach is more flexible and allows for a more traditional Linux experience, but requires more effort to set up and maintain. This approach will be described as a 'traditional' installation going forward and called out where possible.

The process of generating a system image for a Gentoo installation on the VisionFive 2 may be broadly described as follows:

* Gather the installation files and generate a cross-compiler
  * Check out the VisionFive 2 SDK OR kernel sources (until the upstream kernel has full hardware support)
  * Build a cross-compiler and use it to build a kernel, initramfs, and FIT image
* Generate a Gentoo rootfs
  * Unpack and customize a Gentoo Stage 3 tarball to create a Gentoo rootfs
  * Use Catalyst to generate an image from scratch
* Load the Flattened Image Tree (FIT) onto the device
* Write a Gentoo rootfs to the selected storage medium.

Generate a cross-compiler
-----------------------

A cross-compiler will typically also be required as the VisionFive 2 is a RISC-V device and most users will be running an x86_64 (amd64) host.

### Check out the VisionFive 2 BSP

The VisionFive 2 Board Support Package (BSP) (sometimes called an SDK by manufacturers) is a git repository containing a collection of scripts (and git submodules) that may be used to bootstrap a cross-compiler and build a Linux kernel, initramfs, rootfs, U-Boot, and FIT image for the VisionFive 2.

As most consumers of this article already use Gentoo, this step is not essential; Gentoo users have the option of using crossdev (from sys-devel/crossdev) to get an up-to-date riscv64 cross-compiler with which to build the kernel, initramfs, and FIT image. However, the SDK is still useful: it provides a convenient way to build U-Boot and a rootfs, contains a great deal of information about how the developers intend for images to be written to the device, and, for inexperienced embedded users, provides a way to generate a guaranteed bootable image and a less-complex approach which be preferable.

The VisionFive 2 BSP repository is available at: https://github.com/starfive-tech/VisionFive2

Use the following commands to check out the repository:

```bash
git clone https://github.com/starfive-tech/VisionFive2.git
cd VisionFive2
git checkout --track origin/JH7110_VisionFive2_6.6.y_devel
git submodule update --init --recursive
```

This will take some time and require around 9GB of disk space. Some modules may fail because certain dependencies don't have the best git hosting. The only solution is to wait and try again later (or ask someone for a copy of that source repository).

For user who build the release tag version, the above command is enough. For developer, need to switch the 5 submodules buildroot, u-boot, linux, opensbi, soft_3rdpart to correct branch manually, or refer to .gitmodule

```bash
cd buildroot
git checkout --track origin/JH7110_VisionFive2_devel
cd ..
cd u-boot
git checkout --track origin/JH7110_VisionFive2_devel
cd ..
cd linux
git checkout --track origin/JH7110_VisionFive2_6.6.y_devel
cd ..
cd opensbi
git checkout master
cd ..
cd soft_3rdpart
git checkout JH7110_VisionFive2_devel
cd ..
```

### Quick Build Instructions

Below are the quick building for the initramfs image image.fit which could be translated to board through tftp and run on board. The completed toolchain, u-boot-spl.bin.normal.out, visionfive2_fw_payload.img, image.fit will be generated under work/ directory. The completed build tree will consume about 18G of disk space.

```bash
make -j$(nproc)
```

The output of this process (kernel, initramfs, device tree blobs [dtb], OR the FIT image containing them) may be used to boot a Gentoo rootfs; if this is all that is desired there is no need to build custom Gentoo versions.

--- To Be Creating --
