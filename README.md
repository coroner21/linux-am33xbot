# linux-am33xbot
Arch Linux ARM Kernel with botic patches (for Beaglebone Black / Wireless). Please refer to [DiyAudio thread](https://www.diyaudio.com/forums/twisted-pear/258254-support-botic-linux-driver.html) for more information on the driver and its development and checkout Cronus and Hermes-BBB from [Twistedpearaudio](http://twistedpearaudio.com/landing.aspx) for the background on the required cape for the Beaglebone board. Credit for the development of the cape and driver to Russ White from TPA and [Miero](https://github.com/miero).

## Installation instructions
Follow these steps to build a boticized kernel for recent Arch Linux ARM:

1. Make sure you have either a VM or a native install of Arch Linux (**x86-64**) and that you are able to build packages: `pacman -S --needed base-devel`
2. Install the cross-compiler required to build the kernel for the armv7h architecture (AUR: [arm-linux-gnueabihf-gcc-linaro-bin](https://aur.archlinux.org/packages/arm-linux-gnueabihf-gcc-linaro-bin/))
3. Clone this repository, enter the created folder and delete the `.git`hidden folder created by the git tool (otherwise some of the patches will not apply)
4. Run `CARCH=armv7h makepkg -A` in the created folder to build the package
5. Copy the created package (pkg.tar.xz file) to the BBB / BBBW
6. Uninstall the standard kernel package (`pacman -R linux-am33x`). ATTENTION: You need to install the new kernel before rebooting / power off
7. Install the package with `pacman -U`

Before rebooting ensure that Arch Linux ARM is configured to load the required botic overlay and edit /boot/boot.txt accordingly, see following example (which disables also the onboard oscillator):
```
# After modifying, run ./mkscr

if test -n ${distro_bootpart}; then setenv bootpart ${distro_bootpart}; else setenv bootpart 1; fi
part uuid ${devtype} ${devnum}:${bootpart} uuid

setenv bootargs "console=tty0 console=${console} root=PARTUUID=${uuid} rw rootwait"

if load ${devtype} ${devnum}:${bootpart} ${kernel_addr_r} /boot/zImage; then
  gpio set 54
  echo fdt: ${fdtfile}
  if load ${devtype} ${devnum}:${bootpart} ${fdt_addr_r} /boot/dtbs/${fdtfile}; then
    gpio set 55

    fdt addr ${fdt_addr_r}
    # Remove the internal clock as well as the original mcasp pinctrl bindings
    fdt rm /ocp/l4_wkup@44c00000/scm@210000/pinmux@800/mcasp0_pins
    fdt rm /clk_mcasp0_fixed
    fdt rm /clk_mcasp0
    
    # Enable Botic overlay
    # Use below line only if you have ES9018 DAC connected to the I2C header of the Hermes-BBB module:
    #load ${devtype} ${devnum}:${bootpart} 0x88060000 /lib/firmware/BOTIC-SABRE32-00A0.dtbo
    
    # In case you want to use ES9018K2M based DAC, you can use the following overlay line:
    #load ${devtype} ${devnum}:${bootpart} 0x88060000 /lib/firmware/BOTIC-ES9018K2M-00A0.dtbo
    
    # For some generic I2S/DSD DAC connected to the Cronus board you can use the following dummy codec overlay.
    # You can still use I2C directly from the isolated headers of the Cronus board to control the DAC if necessary.
    load ${devtype} ${devnum}:${bootpart} 0x88060000 /lib/firmware/BOTIC-00A0.dtbo
    
    fdt resize ${filesize}
    fdt apply 0x88060000
    
    if load ${devtype} ${devnum}:${bootpart} ${ramdisk_addr_r} /boot/initramfs-linux.img; then
      gpio set 56
      bootz ${kernel_addr_r} ${ramdisk_addr_r}:${filesize} ${fdt_addr_r};
    else
      gpio set 56
      bootz ${kernel_addr_r} - ${fdt_addr_r};
    fi;
  fi;
fi
```

Afterwards run `./mkscr` in the `/boot` directory and reboot. Enjoy the boticized kernel and listen to some good music!

## Instructions in case of non-default clock frequencies

Note that the above instructions only work in case of standard default clock frequencies for the Cronus clock module: 48kHz multiples are assumed to run using a 49152000Hz clock and 44.1kHz multiples are assumed to run using a 45158400Hz clock.

If you have mounted clocks with frequencies different from the above you have to manually edit the dts source file of the overlay you want to use.

Instead of step 4 above, perform the following:
- Run `CARCH=armv7h makepkg -oA` to extract the sources and enter the `src/bb.org-overlays/src/arm` directory
- Edit the `BOTIC-XXX.dts` file to change the frequencies of the two clock definitions `clk48` and `clk44`
- Go back to the root folder of the repository (`linux-am33xbot`) and run `CARCH=armv7h makepkg -eA` to build the package without extracting the sources again.
