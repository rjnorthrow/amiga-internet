# Amiga-Internet

With the appropriate USB to serial cable, allow any Amiga with a working serial port and the appropriate TCP/IP software installed to access the internet or your local area network using any Raspberry Pi.

## Getting Started

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

## Run the Ansible Script
`ansible-playbook amiga_ppp.yml`

### Change the defaults

There are several variables that you can change from the command line, for example if your LAN network clashes with the one set up as the default for the Amiga to Raspberry Pi link (10.168.0.0/24)

* Change the Amiga<->Raspberry Pi network *
`ansible-playbook --extra-vars='serial_ip=192.168.0.1 amiga_ip=192.168.0.2' ppp_amiga.yml`
