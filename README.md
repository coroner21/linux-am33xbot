**Instructions on building a kernel for debian buster based images (to be executed on a debian x86-64 machine with ARM cross compiler installed)

1. Clone the debian-buster branch of this repository with git ``git clone -b debian-buster https://github.com/coroner21/linux-am33xbot.git`` and enter the created directory
2. Clone the official beagleboard.org kernel repositories (4.19 branch) ``git clone -b 4.19 --depth 1 https://github.com/beagleboard/linux.git``
3. Copy the config file to the kernel directory ``cp config linux/.config``
4. Patch kernel sources with botic patches ``cd linux``, ``patch -p1 < ../mcasp_dsd.patch``, ``patch -p1 < ../new_botic_card_codec.patch``
5. Generate a debian binary kernel package ``make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- bindeb-pkg``
6. The resulting linux-image package can be installed into a running debian buster beaglebone image (overwriting the existing kernel image)
7. If you need to build additional out-of-tree kernel modules just install the linux-headers package in addition
8. Copy both ``.dtbo`` device tree overlay files on the beaglebone image into the ``/lib/firmware`` folder
9. Be sure to edit ``/boot/uEnv.txt`` so that the required overlay is loaded. Choose ``BOTIC-00A0.dtbo`` for a generic I2S DAC and ``BOTIC-SABRE32-00A0.dtbo`` for an ES9018 DAC with I2C controls via alsamixer

Additional instructions on using the botic modules can be found in the README text file.
