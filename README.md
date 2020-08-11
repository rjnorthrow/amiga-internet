* Amiga-Internet *

With the appropriate USB to serial cable, allow any Amiga with a working serial port and the appropriate TCP/IP software installed to access the internet or your local area network using any Raspberry Pi.

* Getting Started *

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

* Modify the Ansible Configuration as Required *

Edit `host_vars/localhost/vars` and change the configuration as required

* Run the Ansible Script *
`ansible-playbook amiga_ppp.yml`
