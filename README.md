# ubuntu-pi-image

**If you just want to download Ubuntu MATE images for the Raspberry Pi the go to the Ubuntu MATE website:**

  * **https://ubuntu-mate.org/raspberry-pi/**

These "docs" are not comprehensive, but should be enough to get you started with
building your own Ubuntu based images for the Raspberry Pi.


## Building Images

  * Clone the Retro Home project
    * `git clone https://github.com/wimpysworld/ubuntu-pi-image.git`

It is best to run the `ubuntu-pi-image` on an Ubuntu 22.04 x86 64-bit
workstation, ideally running in a VM via [Quickemu](https://github.com/quickemu-project/quickemu).
If using a fresh [Quickemu](https://github.com/quickemu-project/quickemu) VM
you'll need to set the `disk_size` parameter large enough to complete the build.
This can be achieved by adding `disk_size="64G"` to `ubuntu-mate-jammy.conf`
before running `quickemu` to create the VM. Alternatively you could mount
external storage into the container for the build area. You'll also need at
least to `sudo apt install git`.

## apt-cacher-ng

If you `apt-get install apt-cache-ng` the `ubuntu-pi-image` script will
automatically use the apt-cache-ng proxy to accelerate build performance.

## Build configuration

You can tweak some variables towards the bottom of the `ubuntu-pi-image` script.

```bash
FLAVOUR="ubuntu-mate"
IMG_QUALITY="-beta1"
IMG_VER="22.04"
IMG_RELEASE="jammy"
IMG_ARCH="arm64"
```

### Usage

The following incantation will build a Ubuntu MATE 22.04 arm64 image for
Raspberry Pi.

```bash
sudo ./ubuntu-pi-image
```

The script will create `~/Build` in your home directory that will include the
caches for each stages and the image for putting on your Raspberry Pi SD card,
USB stick or SSD.

## Other Desktop

Modify the `stage_02_desktop` and `stage_03_snap` accordingly to adapt the script
for other Ubuntu flavours/desktops.

## Bootloaders

This is the bootloader configuration from Ubuntu 22.04.

### cloud-init

Remove these files:

  * `meta-data`
  * `network-config`
  * `user-data`
### README

```
An overview of the files on the /boot/firmware partition (the 1st partition
on the SD card) used by the Ubuntu boot process (roughly in order) is as
follows:

* bootcode.bin   - this is the second stage bootloader loaded by all pis with
                   the exception of the pi4 (where this is replaced by flash
                   memory)
* config.txt     - the configuration file read by the boot process
* start*.elf     - the third stage bootloader, which handles device-tree
                   modification and which loads...
* vmlinuz        - the Linux kernel
* cmdline.txt    - the Linux kernel command line
* initrd.img     - the initramfs
* meta-data      - meta-data for cloud-init; usually just contains the
                   instance id
* network-config - network configuration for cloud-init; edit this to set up
                   wifi access points and other networking settings
* user-data      - user-data for cloud-init; edit this to configure initial
                   users, SSH keys, packages, etc.
```
### 32bit server

From the armhf server image.
#### cmdline.txt
```
console=serial0,115200 dwc_otg.lpm_enable=0 console=tty1 root=LABEL=writable rootfstype=ext4 rootwait fixrtc quiet splash
```

#### config.txt
```
[all]
kernel=vmlinuz
cmdline=cmdline.txt
initramfs initrd.img followkernel

[pi4]
max_framebuffers=2
arm_boost=1

[all]
# Enable the audio output, I2C and SPI interfaces on the GPIO header. As these
# parameters related to the base device-tree they must appear *before* any
# other dtoverlay= specification
dtparam=audio=on
dtparam=i2c_arm=on
dtparam=spi=on

# Comment out the following line if the edges of the desktop appear outside
# the edges of your display
disable_overscan=1

# If you have issues with audio, you may try uncommenting the following line
# which forces the HDMI output into HDMI mode instead of DVI (which doesn't
# support audio output)
#hdmi_drive=2

# Enable the serial pins
enable_uart=1

# Autoload overlays for any recognized cameras or displays that are attached
# to the CSI/DSI ports. Please note this is for libcamera support, *not* for
# the legacy camera stack
camera_auto_detect=1
display_auto_detect=1


[cm4]
# Enable the USB2 outputs on the IO board (assuming your CM4 is plugged into
# such a board)
dtoverlay=dwc2,dr_mode=host

[all]
```

### 64bit server

From the arm64 server image.
#### cmdline.txt
```
console=serial0,115200 dwc_otg.lpm_enable=0 console=tty1 root=LABEL=writable rootfstype=ext4 rootwait fixrtc quiet splash
```

#### config.txt
```
[all]
kernel=vmlinuz
cmdline=cmdline.txt
initramfs initrd.img followkernel

[pi4]
max_framebuffers=2
arm_boost=1

[all]
# Enable the audio output, I2C and SPI interfaces on the GPIO header. As these
# parameters related to the base device-tree they must appear *before* any
# other dtoverlay= specification
dtparam=audio=on
dtparam=i2c_arm=on
dtparam=spi=on

# Comment out the following line if the edges of the desktop appear outside
# the edges of your display
disable_overscan=1

# If you have issues with audio, you may try uncommenting the following line
# which forces the HDMI output into HDMI mode instead of DVI (which doesn't
# support audio output)
#hdmi_drive=2

# Enable the serial pins
enable_uart=1

# Autoload overlays for any recognized cameras or displays that are attached
# to the CSI/DSI ports. Please note this is for libcamera support, *not* for
# the legacy camera stack
camera_auto_detect=1
display_auto_detect=1

# Config settings specific to arm64
arm_64bit=1
dtoverlay=dwc2

[cm4]
# Enable the USB2 outputs on the IO board (assuming your CM4 is plugged into
# such a board)
dtoverlay=dwc2,dr_mode=host

[all]
```

### Desktop

From Ubuntu proper desktop image.

#### cmdline.txt
```
zswap.enabled=1 zswap.zpool=z3fold zswap.compressor=zstd dwc_otg.lpm_enable=0 console=tty1 root=LABEL=writable rootfstype=ext4 rootwait fixrtc quiet splash
```
### config.txt
```
[all]
kernel=vmlinuz
cmdline=cmdline.txt
initramfs initrd.img followkernel

[pi4]
max_framebuffers=2
arm_boost=1

[all]
# Enable the audio output, I2C and SPI interfaces on the GPIO header. As these
# parameters related to the base device-tree they must appear *before* any
# other dtoverlay= specification
dtparam=audio=on
dtparam=i2c_arm=on
dtparam=spi=on

# Comment out the following line if the edges of the desktop appear outside
# the edges of your display
disable_overscan=1

# If you have issues with audio, you may try uncommenting the following line
# which forces the HDMI output into HDMI mode instead of DVI (which doesn't
# support audio output)
#hdmi_drive=2

[cm4]
# Enable the USB2 outputs on the IO board (assuming your CM4 is plugged into
# such a board)
dtoverlay=dwc2,dr_mode=host

[all]

# Enable the KMS ("full" KMS) graphics overlay, leaving GPU memory as the
# default (the kernel is in control of graphics memory with full KMS)
dtoverlay=vc4-kms-v3d

# Autoload overlays for any recognized cameras or displays that are attached
# to the CSI/DSI ports. Please note this is for libcamera support, *not* for
# the legacy camera stack
camera_auto_detect=1
display_auto_detect=1

# Config settings specific to arm64
arm_64bit=1
dtoverlay=dwc2
```
