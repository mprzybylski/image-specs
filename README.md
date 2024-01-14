# Raspberry Pi image specs

This repository contains the files with which the images referenced at
https://wiki.debian.org/RaspberryPiImages have been built.

## Option 1: Downloading an image

See https://wiki.debian.org/RaspberryPiImages for where to obtain the
latest pre-built image.

## Option 2: Building your own image

If you prefer, you can build a Debian Raspberry Pi image
yourself. If you are reading this document online, you should first
clone this repository:

```shell
git clone --recursive https://salsa.debian.org/raspi-team/image-specs.git
cd image-specs
```

For this you will first need to install the following packages on a
Debian Bullseye (11) or higher system:

* binfmt-support
* bmap-tools
* debootstrap
* dosfstools
* fakemachine (optional, only available on amd64)
* kpartx
* qemu-utils
* qemu-user-static
* time
* vmdb2 (>= 0.17)
* python3
* zerofree (because of [#1021341](https://bugs.debian.org/1021341))

To install these (as root):
```shell
   apt install -y vmdb2 dosfstools qemu-utils qemu-user-static debootstrap binfmt-support time kpartx bmap-tools python3 zerofree
   apt install -y fakemachine
```

If debootstrap still fails with exec format error, try
running `dpkg-reconfigure qemu-user-static`. This calls
`/var/lib/dpkg/info/qemu-user-static.postinst` which uses binfmt-support
to register the executable format with /usr/bin/qemu-$fmt-static

This repository includes a master YAML recipe (which is basically a
configuration file) for all of the generated images, diverting as
little as possible in a parametrized way. The master recipe is
[raspi_master.yaml](raspi_master.yaml).

A Makefile is supplied to drive the build of the recipes into images.
If `fakemachine` is installed, it can be run as an unprivileged user.
Otherwise, because some steps of building the image require root privileges,
you'll need to execute `make` as root.

The argument to `make` is constructed as follows:
`raspi_<model>_<release>.<result-type>`

Whereby <model\> is one of `1`, `2`, `3` or `4`, <release\> is either
`bullseye`, `bookworm`, or `trixie`; and <result-type\> is `img` or `yaml`.

Model `1` should be used for the Raspberry Pi 0, 0w and 1, models A and
B. Model `2` for the Raspberry Pi 2 models A and B. Model `3` for all
models of the Raspberry Pi 3 and model `4` for all models of the
Raspberry Pi 4.
So if you want to build the default image for a Raspberry Pi 3B+ with
Bullseye, you can just issue:

```shell
   make raspi_3_bullseye.img
```

This will first create a `raspi_3_bullseye.yaml` file and then use that
*yaml* recipe to build the image with `vmdb2`.

You can also edit the `yaml` file to customize the built image. If you
want to start from the platform-specific recipe, you can issue:

```shell 
make raspi_3_bullseye.yaml 
``` 
The recipe drives [vmdb2](https://vmdb2.liw.fi/), the successor to
`vmdebootstrap`. Please refer to [its
documentation](https://vmdb2.liw.fi/documentation/) for further details;
it is quite an easy format to understand.

Copy the generated file to a name descriptive enough for you (say,
`my_raspi_bullseye.yaml`). Once you have edited the recipe for your
specific needs, you can generate the image by issuing the following (as
root):

```shell
vmdb2 --rootfs-tarball=my_raspi_bullseye.tar.gz --output \
my_raspi_bullseye.img my_raspi_bullseye.yaml --log my_raspi_bullseye.log
```

This is, just follow what is done by the `_build_img` target of the
Makefile.

## Installing the image onto the Raspberry Pi

Plug an SD card which you would like to entirely overwrite into your SD card reader.

Assuming your SD card reader provides the device `/dev/mmcblk0`
(**Beware** If you choose the wrong device, you might overwrite
important parts of your system.  Double check it's the correct
device!), copy the image onto the SD card:

```shell
bmaptool copy raspi_3_bullseye.img.xz /dev/mmcblk0
```

Alternatively, if you don't have `bmap-tools` installed, you can use
`dd` with the compressed image:

```shell
xzcat raspi_3_bullseye.img.xz | dd of=/dev/mmcblk0 bs=64k oflag=dsync status=progress
```

Or with the uncompressed image:

```shell
dd if=raspi_3_bullseye.img of=/dev/mmcblk0 bs=64k oflag=dsync status=progress
```

Then, plug the SD card into the Raspberry Pi, and power it up.

The image uses the hostname `rpi0w`, `rpi2`, `rpi3`, or `rpi4` depending on the
target build. The provided image will allow you to log in with the
`root` account with no password set, but only logging in at the
physical console (be it serial or by USB keyboard and HDMI monitor).

# Print server next steps
* SSH into the Pi at its static address.  (You _did_ configure public key auth for the root account, right?)
* Update the root password.
* Connect the printer via usb
* Use `lpinfo -v` to get the device URI
* Use `lpadmin -p InLivingColor -m /path/to/ppd -v usb://Mfr/Device/Uri` to create a printer called "InLivingColor".
* Use `lpoptions -l InLivingColor` to dump settable options for the printer like paper size, duplexing, etc.
* i.e...
  ```text
  lpadmin -p InLivingColor -m lsb/usr/custom/Xerox_Phaser_6280DN.ppd -v usb://Xerox/Phaser%206280DN?serial=NKA101018 \
    -o InstalledMemory=256Meg \
    -o Option1=None \
    -o Option2=False \
    -o Duplex=DuplexNoTumble \
    -o PageSize=Letter
  cupsenable InLivingColor # To enable the printer
  cupsaccept InLivingColor # To start accepting jobs for it
  ```
* Use `lp -d InLivingColor - <<< "Hello World."` to print a simple, local test page.

# To do:
* Create the avahi service file to allow AirPrint. Reference:
  https://www.linuxbabe.com/ubuntu/set-up-cups-print-server-ubuntu-bonjour-ipp-samba-airprint
  https://www.cups.org/doc/spec-ipp.html looks like the reference for `printer-type`