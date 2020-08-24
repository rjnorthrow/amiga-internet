# Amiga-Internet

With the appropriate USB to serial cable [like this one](https://plugable.com/products/pl2303-db9/), allow any Amiga with at least 1MiB RAM, Kickstart version 37 or higher, a working serial port and the [appropriate TCP/IP software](http://aminet.net/comm/net/AmiTCP-bin-30b2.lha) installed to access the internet or your local area network using any Raspberry Pi.

## Getting Started (Raspberry Pi)

Install Raspberry Pi OS, you only need the 'lite' version without a GUI

Log in and type the following into your terminal to update your Pi to the latest operating system version and reboot afterwards:
```
sudo apt update && sudo apt dist-upgrade -y && sudo reboot
```

Log back in and tidy up after reboot:
`sudo apt autoremove -y && sudo apt autoclean`

Install ansible and git:
`sudo apt install -y ansible git`

Fetch the ansible repo from GitHub:
`git clone https://github.com/rjnorthrow/amiga-internet.git`

Change to the `amiga-internet` directory

`cd amiga-internet`

## Install the required `rjnorthrow.ppp_amiga` role
`ansible-galaxy install rjnorthrow.ppp_amiga`

## Run the Ansible Script
`ansible-playbook amiga_ppp.yml`


## Getting Started (Amiga)

Any Amiga will do, with a working serial port

### Install AmiTCP

Firstly, make a `New Drawer` on your hard disk to contain the software, for example `Internet`. I put this on my `System` or `Workbench` partition

Getting the software on to your Amiga may be the tricky bit. I used a Gotek drive, copying the software on to a virtual floppy disk on a USB key with [ADF Opus](http://adfopus.sourceforge.net/). You can also use [Amiga Explorer](https://www.amigaforever.com/ae/) as you already have the serial cable

You may also need to copy the [LhA](http://aminet.net/util/arc/lha.run) software in order to unpack `AmiTCP-3.0b2.lha`. Copy this to your Amiga into the `Ram Disk`. Run this from the `Ram Disk` on your Amiga and rename one of the extracted files which matches your processor. For example, rename `lha_68020` as `lha` if you are running on an Amiga 1200. Finally, copy this file to `SYS:C`

You can now extract the `AmiTCP-3.0b2.lha` file. Copy it again to your `Ram Disk` if you have enough RAM (~2.8MiB spare) or to a hard disk partition such as `WORK:`, open a `Shell` and type:
```
RAM:
lha x amitcp#?
```
The files will be extracted into a directory called `AmiTCP-3.0b2`. You can now run the installer `Install_AmiTCP`

When asked, choose the following options:
* `Intermediate User`
* `Install for Real`
* Select directory where to install AmiTCP/IP: `SYS:Internet/AmiTCP-3.0b2`
* Do you want to install example Sana-II configuration files? `Yes`
* Select Sana-II configuration files to be copied: `Proceed with Copy` - you don't need these but it creates the directory for you
* Do you want to install Napsaterm fonts, napsa? `No`
* Logging in as 'root' and changing the root's password: `Proceed`
* Log in as super-user (root), Password: Just press Return, we can't log in at this stage
* Login incorrect: There is a bug in the installation script, simply close the window
* Enter the default user name: Put anything in here, such as your first name
* Enter the host name of your computer: Put anything in here, such as `Amiga`
* Enter the domain part of your host name: Put anything in here, such as `example.com`
* The host name will be stored to the environment variable HOSTNAME: `Store to ENV(ARC)` or `Use "setenv"`, it doesn't matter
* Give aliases to your computer: `Proceed`
* Select a SANA-II device driver: `ppp.device` - not listed? Download and install [this](http://m68k.aminet.net/comm/net/PPP1_45.lha)
* Select unit number: `0`
* IP address: `10.168.0.2`
* Give the destination address: `10.168.0.1`
* Netmask of network: `255.255.255.254`
* Is this correct? `Yes`
* Select a SANA-II device driver: `Proceed`
* Enter the IP address of the default gateway: `10.168.0.1`
* Give domain names (one at a time) to search: `Proceed`
* Give domain names (one at a time) to search: `Proceed`
* Enter the IP addresses of the name servers (one at a time): Use your LAN's DNS resolvers, it might be `192.168.0.1` for example
* Enter the IP addresses of the name servers (one at a time): `Proceed`
* Do you want the AmiTCP/IP to be started at the system startup? `Yes` or `No` depending on your preference
* Do you want Installer to make the required changes to your s:user-startup script? `Yes` (probably!)
* Do you want the Inetd to be started at the AmiTCP/IP startup? `No`
* (Read the README)
* Installation complete! `Proceed`

At this point, the software is installed but not configured properly. Take the following steps to configure the software by opening a `Shell`:

`ed S:user-startup` - Remove the whole line starting `AmiTCP:bin/login` as this won't work until the TCP/IP stack is running

`ed ENVARC:sana2/ppp0.config` - Add the line: `serial.device 0 57600 0.0.0.0`, this configures `ppp0` to use the serial interface

`ed SYS:Internet/AmiTCP-3.0b2/db/interfaces` - Add the line: `ppp DEV=Devs/Networks/ppp.device` to bind `ppp0` to the `ppp` device

`ed SYS:Internet/bin/startnet`:
* Add a new line at the top: `online Devs:Networks/ppp.device 0`, this brings up the `ppp0` interface, configured above
* Change `run AmiTCP:AmiTCP` to `run >NIL: AmiTCP:AmiTCP` to discard output from the command
* Change `AmiTCP:bin/ifconfig lo/0 localhost` to `ifconfig lo0 localhost`, fixing the device name, and ifconfig is already in our path
* Change `AmiTCP:bin/ifconfig Devs:Networks/ppp.device/0` to `ifconfig ppp0`, keeping the rest of the line

Now ensure that `pppd` is running on your Raspberry Pi and then reboot your Amiga

In order to tidy up the connection after use, type: `offline Devs:Networks/ppp.device 0`. This makes sure that the Raspberry Pi end is informed that the connection has been terminated and it will restart the PPP daemon so that you can connect again


## Troubleshooting

There are several variables that you can change from the command line, for example if your LAN network clashes with the one set up as the default for the Amiga to Raspberry Pi link (10.168.0.0/24)

*Change the Amiga<->Raspberry Pi network*

`ansible-playbook --extra-vars='serial_ip=192.168.0.1 amiga_ip=192.168.0.2' ppp_amiga.yml`

*Use a serial cable connected to the GPIO pins rather than USB*

`ansible-playbook --extra-vars='ppp_serial_device=/dev/ttyAMA0' ppp_amiga.yml`

*If you are using the wireless rather than the Ethernet connector*

`ansible-playbook --extra-vars='interface=wlan0' ppp_amiga.yml`
