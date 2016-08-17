# PIphoto client

[![Join the chat at https://gitter.im/piphoto/piphotoclient](https://badges.gitter.im/piphoto/piphotoclient.svg)](https://gitter.im/piphoto/piphotoclient?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

System for transferring DSLR pictures to a remote host using a Raspberry Pi.

#About
Inspired by the following blogpost [Camera Pi – DSLR Camera with Embedded Computer](http://www.davidhunt.ie/raspberry-pi-in-a-dslr-camera/) I decided to recreate (to some extent) this project using the new Raspberry Pi 3. **PIphoto** allows you to attach your Raspberry PI to your camera (DSLR/Point-and-Shoot) and take pictures in **raw or JPEG** format, resize those images on the fly and send them to a remote system (PC/tablet/server). 

Doesn't this functionality already exist? Yes kinda, there are mobile applications which seem to have this kind of functionality. There are also devices created by camera manufacturers which seem to implement this kind of functionality, for instance the
[wireless file transmitter](https://www.usa.canon.com/internet/portal/us/home/products/details/cameras/wireless-file-transmitter/wireless-file-transmitter-wft-e6a) created by Canon. But one problem with the wireless file transmitter is its hefty price. Expect to pay between 450 and 600 USD. Another problem is that the WiFi transmitter only works over WiFI. In a time where everybody seems to be connected via 3G/4G/LTE its seems more than logical that photographers should be able to use that technology for transferrring their images. The **Raspberry Pi with its built-in WiFi** or connected with a **3G/4G/LTE USB dongle** feels like a perfect candidate to create a truly mobile photography workflow.

#Project dependencies

* [Raspberry Pi (2/3)](https://www.raspberrypi.org/) - The Raspberry Pi is a low cost, credit-card sized computer. It’s capable of doing everything you’d expect a desktop computer to do.
* [microSD](https://simple.wikipedia.org/wiki/MicroSD) - The size of the needed microSD cards depends on the fact if one is shooting in **RAW**, **RAW+JPEG**, or **JPEG** mode only. But for testing purpose a size of **4GB/8GB** should be sufficient.
* [Linux distribution](https://archlinuxarm.org/platforms/armv8/broadcom/raspberry-pi-3) - For this project we are going to use Arch Linux. Although others can be used, e.g. [Rasbian](https://www.raspbian.org/FrontPage).
* [inotify-tools](https://github.com/rvoicilas/inotify-tools/wiki) - inotify-tools is a C library and a set of command-line programs for Linux providing a simple interface to inotify. These programs can be used to monitor and act upon filesystem events.  
* [Elixir programming language](http://elixir-lang.org/) - Elixir is a dynamic, functional language designed for building scalable and maintainable applications.
* [gPhoto2](http://gphoto.sourceforge.net/) - gPhoto2 is a free, redistributable, ready to use set of digital camera software applications for Unix-like systems, written by a whole team of dedicated volunteers around the world. It supports more than 2100 cameras.
* [dcraw](http://www.cybercom.net/~dcoffin/dcraw/) - Decoding raw digital photos in Linux.
* [libvips](http://www.vips.ecs.soton.ac.uk/index.php?title=Libvips) - libvips is a fully demand-driven, threaded image processing library with no image size limits and with good support for colour. It is mature and documented.

#Getting started
First we need to install Archlinux on our Raspberry Pi 3/2. These instructions assume you are installing Archlinux from an existing linux installation. Follow the installation guide on the [archlinux|ARM site](https://archlinuxarm.org/platforms/armv8/broadcom/raspberry-pi-3). Use the serial console or SSH to the IP address give to the board by your router.

* Login as the default user ***alarm*** with the password ***alarm***.
* The default root password is root.

Create a directory to save the images and or video:
```
mkdir -p capture/{received,jpg,raw,video}
```
As root add our user "alarm" to /etc/sudoers file:
```
# echo "alarm ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers

```
As root create the group "capture" and add user alarm to the group "capture":

```
# groupadd capture && gpasswd -a alarm capture

```
Then install the essential packages with the following command:
```
As root issue the following: pacman -Syu sudo inotify-tools elixir gphoto2 dcraw imagemagick libwebp
```
Next we are going to add a udev rule and a systemd unit file. We need these so that the Raspberry Pi 3 recognizes the camera and starts gphoto2 in tethered shooting mode.

```
sudo touch /etc/udev/rules.d/30-capture.rules
```
Now use your favorite text editor (vim/emacs/nano) to add the following:
```
ACTION=="add", SUBSYSTEM=="usb", ATTRS{product}=="Canon Digital Camera", ATTRS{idVendor}=="04a9", GROUP="capture", TAG+="systemd", SYMLINK+="ccapture"
ACTION=="remove", SUBSYSTEM=="usb", ATTRS{product}=="Canon Digital Camera", ATTRS{idVendor}=="04a9"
```
This udev rule adds a symlink named **ccapture** in the /dev directory when a Canon usb device is inserted into the Raspberry PI. This enables us to reference the camera by its symlink. Note that my camera is a Canon. Replace **ATTRS{idVendor}=="04a9"** with **ATTRS{idVendor}=="04b0"** if your camera is a Nikon. 

Now we have to add a device unit for systemd, create a file **/etc/systemd/system/capture.service"** and add the following:
```
[Unit]
Description=Capture with gphoto %i
After=dev-ccapture.device
BindsTo=dev-ccapture.device
Requisite=dev-ccapture.device

[Service]
WorkingDirectory=/home/alarm/capture/received
ExecStartPre=/usr/bin/gphoto2 --set-config capturetarget=1
ExecStart=/usr/bin/gphoto2 --capture-tethered
User=alarm
Restart=always
SuccessExitStatus=1 2 8 9 15 SIGKILL
[Install]
WantedBy=dev-ccapture.device
```

The last step for the basic setup is to reload the udev and systemd:
```
sudo systemctl daemon-reload
sudo systemctl enable capture.service
sudo systemctl start capture.service
```

Connect your camera and take a picture, if everything is correct the photo's should be saved on your device in the **/home/alarm/capture/received** directory.

#Contributing 
Please, do! Check out the latest issues to see what needs being done, or add your own cool thing.

#License
PIphoto client is the client part for transferring DSLR pictures to a remote host using a Raspberry Pi

Copyright (C) 2016 PIphoto

This program is free software: you can redistribute it and/or modify it under the terms of the GNU General Public License as published by the Free Software Foundation, either version 3 of the License, or (at your option) any later version.

This program is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU General Public License 

You should have received a copy of the GNU General Public License along with this program. If not, see http://www.gnu.org/licenses/.
