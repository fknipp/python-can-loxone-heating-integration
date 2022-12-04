# python-can-loxone-heating-integration

Integration od the Elfatherm/Kromschröder heating controller into Loxone using a Raspberry Pi and Python

## Hardware

* [Loxone Miniserver Gen 1](https://www.loxone.com/enen/kb/miniserver-gen-1/)
* [Raspberry Pi 3 Model B](https://www.raspberrypi.com/products/raspberry-pi-3-model-b/)
* [Innomaker USB2CAN](https://www.inno-maker.com/product/usb-can/)
* Kromschröder/Elfatherm E8.034 and E8.1124

## Preparation tasks

### Setting up the Raspberry Pi

Used image: Raspberry Pi OS Lite (64-bit), available on https://www.raspberrypi.com/software/operating-systems/

During the initial boot the keyboard, a user and password is configured. After the first login, the WLAN is set up using `raspi-config`.

The system is updated by

```bash
sudo apt update
sudo apt upgrade
```

### Connecting the heating controller

The USB2CAN interface is connected to the respective bus pins in the heating controller named +, - and L.

The USB2CAN interface is connected to the Raspberry Pi using USB.

### Finding out CAN addresses

The CAN interface needs to be configured by

```
sudo ip link set can0 up type can bitrate 20000 listen-only off
```

The programs built by Jürg Müller are used to scan the CAN bus for valid CAN ids.

The archive is installed by

```sh
wget http://juerg5524.ch/data/can_progs.zip
unzip can_progs.zip
cd can_progs
chmod a+x *.arm
```
The linking errors are fixed by changing the files to compile the programs 
```sh
perl -w -i -n -e 's/-lpthread/-lpthread -pthread/g; print' *.arm
```
The scan utility is built by
```sh
./can_scan.arm
```
The scan is performed by
```sh
stdbuf -oL ./can_scan can0 79f total | tee can_scan.out
```
`tee` is used in combination with `stdbuf` to see the progress and save the output to _can_scan.out_.

### Preparing the Loxone program

## 


