# Starting from scratch
You can do most/all of what follows in the Rpi Imager tool. Annoyingly, there is no way to export the config, you just get the .iso.
I've included it anyway, since its the work I've done to get the audio working and there may be stuff that you can't do from the Imager.

Useful docs for "Setting up a headless Pi"
https://www.raspberrypi.com/documentation/computers/configuration.html#setting-up-a-headless-raspberry-pi




## Flashing OS image

* Download the 64-bit Lite image of Rpi OS.
* Plug in micro-USB cable to the bootloader connector
* Run rpiboot
* rpiboot will talk to the bootloader and establish block device(s) on your machine
* lsblk - you'll see the pi as, eg., /dev/sda
* dd the OS image into /dev/sda. As usual, be DAMNED SURE of the destination device.

## Serial Console

* Remove bootloader USB cable
* Plug in USB cable to the main power and IO Molex ClikMate connector. 
* lsusb - you'll see e.g. ttyUSB0.
* To get a serial console, you need to
  * add enable_uart=1 to /boot/config.txt
  * /boot/cmdline.txt should look like this:
  console=serial0,115200 console=tty1 root=PARTUUID=blah rootfstype=ext4 fsck.repair=yes rootwait
* reboot while watching the serial console. You should see it boot, and you'll get a login prompt. You won't be able to log in until you've created a user.

## Create a user
For first boot, to get a valid user:

"This file should contain a single line of text, consisting of username:password – so your desired username, followed immediately by a colon, followed immediately by an encrypted representation of the password you want to use."

/boot/userconf.txt
buggy:HASH

To get HASH run on another machine:
openssl passwd -6

## Set up other stuff - ssh, hostname, etc.

If you haven't done this stuff from the Imager tool, you should now be able to log in on the serial console and do it by running:
raspi-config

## Building the soundcard driver

* Sanity check:
    arecord -l 
  should report:
    **** List of CAPTURE Hardware Devices ****
  (empty list)

* Install git
* Clone the bugg-soundcard-driver repo

* Maybe not necessary: Instal the kernel headers
  sudo apt install raspberrypi-kernel-headers

* Build the driver
  cd codec
  make
  sudo make install

* Run the sanity check again - it'll still be an empty list.



## Building the Device Tree Overlay

* Again in the bugg-soundcard-driver repo
    cd dt
    dtc -@ -I dts -o dtb -o bugg-i2s-soundcard.dtbo bugg-i2s-soundcard-overlay.dts
  Note that warnings may be produced but the blob may build ok.

* Install the blob:
    sudo cp bugg-i2s-soundcard.dtbo /boot/overlays

* Reboot

* arecord -l now reports:
```
**** List of CAPTURE Hardware Devices ****
card 0: buggmemsmic [bugg-mems-mic], device 0: bcm2835-i2s-ad400x-hifi ad400x-hifi-0 [bcm2835-i2s-ad400x-hifi ad400x-hifi-0]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
```

## Debugging Device Tree

* Enable debug output. To /boot/config.txt add:
  dtdebug=1

* You'll now see debug lines in the boot debug output prefaced with "dtdebug: "


## Installing pip and spidev

sudo apt install python3-pip
sudo pip install spidev


## Enabling the SD card

dtoverlay=sdio,poll_once=off

### Speed test
Random Sandisk 16GB card. 
Create 1gb test file with 
dd if=/dev/urandom of=./1gb bs=1M count=1024

time sudo cp 1gb /mnt/sd/
real	2m5.444s

time md5sum /mnt/sd/1gb 
48037acae8c0a4dd3238841bdba0327d  /mnt/sd/1gb
real	0m55.269s

Sums match ok. Seems fairly slow, though.
Write speed of 1024M / (120+5.4) = 8.17 MB/s

#### On my laptop for comparison
time sudo cp 1gb /mnt/sd/
Executed in  102.36 secs

ok, only 20s faster, so it's the SD card that's the bottleneck, it seems. 

### Enabling the SD card disables Wifi
This is only a concern during testing (as the production units don't have wifi). 

The BCM2711 has 3 SD interfaces:
* EMMC2 - this is connected to the onboard eMMC, so we can't touch it. Supports SDIO.
* SDHCI - the other SDIO interface - by default, connects to wifi. Can be re-muxed to SDIO on GPIO's. e.g., for an SD card, as we're doing above
* SDHOST - older device, doesn't support full SDIO, but can be muxed to GPIO's 22-27 (which we're using for the SD slot). 

So we have two options:
1) Accept the default, use SDHCI for the SD card, and have no wifi possibility during development, or in future products
2) Switch the SD card over to using the legacy SDHOST interface.

Option 2 seems preferable, but it requires a custom device tree overlay.
https://forums.raspberrypi.com/viewtopic.php?t=288772
https://forums.raspberrypi.com/viewtopic.php?t=335417

Ok, implemented option 2. It works, performance is the same as with SDHCI.

