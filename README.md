# Tutorial: Raspberry Pi Zero W headless setup + using it as a airplay speaker

* Most of this guide can be used to on a Raspberry Pi 3 to.
* This tutorial is a combination of four different guides/tutorials I used to get my Pi up and running, links in the bottom.
* Feel free to point out errors, typos or improvements, feedback is overall appreciated.

---

#### Shopping list

* Raspberry pi zero or Raspberry pi zero W
* USB adapter (5v) and cable (micro B to USB).
* Micro USB WiFi Dongle, this is only needed if you bought the ** raspberry pi zero ** and not the **raspberry pi zero W** (which has a wifi adapter build in)
* USB Sound Adapter, (USB to 3.5mm converter)
* USB to microUSB Converter (only needed if the sound adapter isn't usb micro)
* 8gb SD card (would recommend class 6 or higher)
* A case to the raspberry (if you prefer)
* A speak of some sort that has a 3.5 mm audio jack

---

### 1. Preparation

* Grab the latest [Raspbian image](https://www.raspberrypi.org/downloads/).
  Raspbian stretch lite is usually preferred for headless, but either version will work (don't use NOOBS).
* Get the [Etcher](https://etcher.io/) application and use that to write the image to your SD card (no need to extract it from the archive, just write it).
* Once the image is written, open the command line and enter:

  `ls /Volumes` You should see a SD card called `boot`.

* Access it:

  `cd /Volumes/boot`

* Create two files called `ssh` and `wpa_supplicant.conf`, they are referring to the tiny FAT32 partition (not a folder named boot), prefer using command line tool?

  `touch ssh wpa_supplicant.conf`

* Add into `wpa_supplicant.conf` and replace **ssid**, **psk** and **key_mgmt** with your network settings (**country** setting doesn't really matter, I just kept the default).

  ```
  ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
  update_config=1
  country=US

  network={
  	ssid="your-network-service-set-identifier"
  	psk="your-network-WPA/WPA2-security-passphrase"
  	key_mgmt=WPA-PSK
  }
  ```

* The `ssh` file will be empty (more info in the bottom :point_down:).

### 2. Installation

* Remove the sd card from the computer
* Put into the raspberry pi and connect it to power
* Wait 1-5 minutes so the pi has booted up
* the default hostname for a Raspberry Pi is **raspberrypi** and on most networks you can SSH directly to this instead of the IP address.

  `ssh pi@raspberrypi.local` _The default password is “raspberry”._

  :x: Can't ssh into the raspberry? Check the list of connected devices on your Wi-Fi network to get the raspberry ip instead, example **ssh pi@127.0.0.1** (still same password as before).

  :x: You've gotten the ip but still can't ssh in? You might need to re-add the `ssh` file again onto the SD card.

  * Turn of the raspberry
  * put the sd card back into your computer
  * cd into the SD card as before
  * add the `ssh` file again (why re-add? More in in the bottom :point_down:)
  * start over from top of **2. Installation**.

### 3. Update/Config

_**NOTE:** run these as root (or prefixed with sudo)_

`sudo su` super user /root

* update software running on your Zero (requires git and sudo apt-get).

  `apt-get update -y`

  `apt-get upgrade -y`

* Install required packages.

  `apt-get install alsa-utils autoconf libtool libdaemon-dev libasound2-dev libpopt-dev libconfig-dev avahi-daemon libavahi-client-dev libssl-dev`

* Grab the airplay software

  `git clone https://github.com/mikebrady/shairport-sync.git`

* Build it

  `cd shairport-sync`

  `autoreconf -i -f`

  `./configure --with-alsa --with-avahi --with-ssl=openssl --with-metadata --with-systemd`

  `make`

* Create a user account for it and add it to the audio group

  `groupadd -r shairport-sync`

  `useradd -r -M -g shairport-sync -s /usr/bin/nologin -G audio shairport-sync`

* install and enable service

  `make install`

  `systemctl enable shairport-sync`
  **or**
  `sudo systemctl enable shairport-sync`

* Edit shairport config to add the usb sound card

  `cd /usr/local/etc/`

  `nano shairport-sync.conf`

  ```
  alsa = {
    output_device = "hw:1";
    mixer_control_name = "Speaker";
  };
  ```

  or if you have used this guide for a 'normal' raspberry and want to use the default headphone jacket

  ```
  alsa = {
  output_device = "hw:0";
  mixer_control_name = "PCM";
  };
  ```

* start shariport without reboot

  `service shairport-sync start`

* Plug in the usb sound card

* Reboot the raspberry pi

  `sudo systemctl reboot`

* If you starts Spotify or similar app, the pi should now be available as a device.

* And you're done :white_check_mark:

---

###### Useful pi commands

* `sudo systemctl` [plus one of the following]
  * `poweroff`
  * `reboot`
  * `halt`
  * `shutdown`

###### Why the empty ssh file?

> `ssh` is disabled by default for security reasons. To enable it, create a file named "ssh" or "ssh.txt" in the small FAT32 boot partition of the SD card. The file doesn't need to have anything in it, it just needs to be present. When Raspbian sees it, it will enable SSH and delete the file.

###### Thanks to David Maitland, Rui Carmo, [Mike Brady](https://github.com/mikebrady), HawaiianPi and the raspberry pi community.

###### Guides/tutorials I used

* [Raspberry Pi Zero Headless Setup](https://medium.com/@DavidMaitland/raspberry-pi-zero-headless-setup-92fb72daf88d)
* [Using a raspberry pi as an airplay speaker](https://taoofmac.com/space/blog/2016/11/27/1945)
* [Shairport Sync documentation](https://github.com/mikebrady/shairport-sync)
* [Headless Raspberry Pi Zero, reply from HawaiianPi](https://www.raspberrypi.org/forums/viewtopic.php?t=194026#p1223245)
