# Piradio [![Build Status](https://travis-ci.org/pirateradiohack/PiRadio.svg?branch=master)](https://travis-ci.org/pirateradiohack/PiRadio)
Piradio powers a streaming internet radio client for the "Pirate Radio" hardware sold by Pimoroni: https://shop.pimoroni.com/products/pirate-radio-pi-zero-w-project-kit
or any kit based on their Phat Beat DAC / Audio Amplifier and a Raspberry Pi.  
It also work with the smaller shim audio amp: https://shop.pimoroni.com/products/audio-amp-shim-3w-mono-amp

Features are the ones provided by [mpd](https://github.com/MusicPlayerDaemon/MPD):
- it retains the radio station that was playing after turning off / on the device and also the volume.
- it has a number of interfaces, currently on this image there is a web interface and a console one.
- managing your radio stations can be done from the interface (`add stream` and delete buttons).
- physical buttons on the Phat Beat do what is expected of them
- LEDs are used as a vumeter

## How to get the image

### building from source
- First clone this repository with `git clone https://github.com/pirateradiohack/PiRadio.git`.  
- Configure your wifi settings: copy the file called `config.example` to `config` and edit this last one. You will see where to enter your wifi name, password and country. All 3 settings are necessary. Your changes to this file will be kept in future updates.
- Optionally configure your radio stations: If you create a file called `my-playlist.m3u` with your own list of internet radio streams, it will be installed.
If not, then you can always add stations in the web interface.
- Then build the image. (You can see the whole guide on the official RaspberryPi repo: https://github.com/RPi-Distro/pi-gen). I find it easier to use docker (obviously you need to have docker installed on your system) as there is nothing else to install, just run one command from this directory: `./build-docker.sh`. That's it. On my computer it takes between 15 and 30 minutes. And at the end you should see something like: `Done! Your image(s) should be in deploy/`  
If you don't see that, it's probably that the build failed. It happens to me sometimes for no reason and I find that just re-launching the build with `CONTINUE=1 ./build-docker.sh` finishes the build correctly.

### Ready-to-flash image
For your convenience you will find the latest image pre-built here: [2021-05-11-Piradio-image.zip](https://github.com/pirateradiohack/PiRadio/releases/download/2021-05-11-PiRadio/2021-05-11-Piradio.zip).


Just flash it and configure your wifi. You can also optionally configure your own radio streams playlist.

The files to edit are:
- wifi: `/etc/wpa_supplicant/wpa_supplicant.conf` (edit this file)
- (optionally) playlist: `/home/pi/.config/vlc/playlist.m3u` (create this file)

You can edit them before or after flashing the image:
- before flashing you can mount the `.img`.  
With a modern operating system you probably just have to click the .img.  
With Linux you can use `kpartx` (from the `multipath-tools` package) to be able to mount the partition directly: `sudo kpartx -a path/to/2019-05-23-Piradio-lite.img` followed by `sudo mount /dev/mapper/loop0p2 tmp/ -o loop,rw`  
(you will need to create the mount directory first and check what loop device you are using with `sudo kpartx -l path/to/2019-05-23-Piradio-lite.img`). Then you can edit the files mentionned above. And `sudo umount tmp`.  
You are safe to flash the image.
- after flashing your operating system probably automounts the partitions.

## Burn the image to a SD card
You should find the newly created image in the `deploy` directory.

### graphically
For a user friendly experience you can try [etcher](https://www.balena.io/etcher/) to flash the image to the SD card.

### manually
On linux (and it probably works on Mac too) an example to get it on the SD card would be:  
`sudo dd bs=4M if=deploy/2019-05-23-Piradio-lite.img of=/dev/mmcblk0 conv=fsync`
(of course you need to replace `/dev/mmcblk0` with the path to your own SD card. You can find it with the command `lsblk -f`)
Those settings are recommended by the RaspberryPi instructions.

## Controlling your radio

The radio is equipped with the necessary buttons to turn it on / off, control the volume up and down, and select the next and previous radio stations. Besides that, you can control it remotely, on the network.

The radio connects to the wifi network that was set in the config file. It should receive an IP address from the Internet router. You need to find this IP address in order to control the radio. There are several ways you can do that:

- For convinience, the radio sets its own local domain. Some devices are compatible with this (Android supposedly works, but iOS does not. Linux should be fine if avahi is installed). So you can try to reach the radio at the address `radio.local`.
- The command `nmap 192.168.1.0/24` can list the devices on your network (adapt the network address based on your computer's IP address, most probably the `1` could be another number). You will see a device named `radio.local` along with its IP address.
- Or you could also check your Internet router to get the list of connected devices and their corresponding IP addresses.

### via web interface
You can control your radio via web interface: try to open `http://radio.local` (or the IP address) in a web browser.

### via ssh with a terminal interface
If you prefer the command line, you can ssh into your radio and then use `ncmpcpp` to get a nice terminal interface (see some screenshots here: https://rybczak.net/ncmpcpp/screenshots/).

### via an application
You can use any `mpd` client you like (a non exhaustive list of applications for various platforms can be found here: https://www.musicpd.org/clients/). If you are asked for the port number, it's the default one, 6600. And for the IP address, it's the same thing as above.

## How it is built
The image is built with the official RaspberryPi.org tool (https://github.com/RPi-Distro/pi-gen) to build a Raspbian lite system with all the software needed
to have a working internet radio stream client. It uses `mpd`.

## Motivation
I wanted to have a software that was easy to install on an embedded kit to make an Internet radio receiver.

## Developers
The interesting bits happen in the `stage2/04-pirate-radio` directory. Where files can be added and then instructions can be set for the building tool to create the final OS image.

If you want to test the image locally, without the need to burn it to an SD card, you can use QEMU to emulate the hardware on your system and then create a virtual machine.
- Make sure you have QEMU installed for the arm architecture, you can test with `qemu-system-arm --version`.
- Set `USE_QEMU` to `"1"` or `true` in the config file and then build your image as usual.
- Download the [Buster kernel](https://github.com/dhruvvyas90/qemu-rpi-kernel/raw/master/kernel-qemu-4.19.50-buster) and the [Device Tree File](https://github.com/dhruvvyas90/qemu-rpi-kernel/raw/master/versatile-pb.dtb) in the `deploy` directory.
- Execute `qemu-system-arm -kernel kernel-qemu-4.19.50-buster -cpu arm1176 -m 256 -M versatilepb -dtb versatile-pb.dtb -no-reboot -serial stdio -net nic -net user,hostfwd=tcp::2222-:22,hostfwd=tcp::2223-:80 -append "root=/dev/sda2 panic=1 rootfstype=ext4 rw" -hda name_of_your_image`

That should boot the image. You can then `ssh` into it on `localhost` on port 2222, and use port 80 on `localhost` port 2223.
This is of limited use since some python libraries (ie: GPIO) won't run on devices other than a Raspberry Pi.


Issues and pull requests are welcome.

