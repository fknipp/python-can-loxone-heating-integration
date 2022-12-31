# python-can-loxone-heating-integration

Integration od the Elster/Elfatherm/Kromschröder heating controller into Loxone using a Raspberry Pi and Python

## Hardware

* [Loxone Miniserver Gen 1](https://www.loxone.com/enen/kb/miniserver-gen-1/)
* [Raspberry Pi 3 Model B](https://www.raspberrypi.com/products/raspberry-pi-3-model-b/)
* [Innomaker USB2CAN](https://www.inno-maker.com/product/usb-can/)
* Elster/Kromschröder/Elfatherm E8.0634 and E8.1124

## Tasks and status

- [x] Set up Raspberry Pi
- [x] Create connection to CAN bus
- [ ] Create virtual inputs in Loxone
- [ ] Send test values to virtual inputs
- [ ] Create mapping of values needed for the visualization
- [ ] Write Python program to read values from CAN bus
- [ ] Extend Python program to send values to Loxone via HTTP

## Preparation tasks

### Set up the Raspberry Pi

Used image: Raspberry Pi OS Lite (64-bit), available on https://www.raspberrypi.com/software/operating-systems/

During the initial boot the keyboard, a user and password is configured. After the first login, the WLAN is set up using `raspi-config`.

The system is updated by

```bash
sudo apt update
sudo apt upgrade
```

### Connect the heating controller

The USB2CAN interface is connected to the respective bus pins in the heating controller named H, L and -. 

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

### Create virtual inputs in Loxone

It's recommended to perform the following steps in Loxone Config:

1. Create a room for the heating. It will be used to restrict permissions on this single room.
1. Create a user for the API calls. The user *heating* will be used in the following example calls.
   * Create and remember a password for this user.
   * Manage the permissions to to give the user access to the newly created room as part of the standard user group.

Afterwards, a virtual input is created for every value to be shown in the user interface. The names of the virtual inputs is later used to send values to these inputs.

1. Select *Virtual Inputs* in the periphery pane.
1. Create a new *Virtual Input* by clicking on the icon in the menu bar.
1. Set the name of the input in the properties pane.
1. Select the previously created room for this input.
1. Unset *Use Digital Input*.
1. Set the unit accordingly, e. g. *<v.1>°C*
1. Set *Show status only*.

Repeat these steps for all virtual inputs.

### Send values to virtual inputs

The function of the virtual inputs can be tested by

    curl -u heating:password http://ip_of_loxone/dev/sps/io/name_of_input/value

e. g.

    curl -u heating:password http://10.0.0.14/dev/sps/io/outside_temperature/12.0


API documentation in German: https://www.loxone.com/dede/kb/webservices/

