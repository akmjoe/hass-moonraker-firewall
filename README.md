# Purpose
I have a power switch for the MCU configured in moonraker, bound to klipper. This means when the power switch is off, so is klipper.

This created a problem with the Home Assistant integration. When Home Assistant attempted to connect to moonraker with klipper shutdown, it would fail. After too many failed attempts, moonraker shuts down. On checking the log, I found  `Maximum Number of Websocket Connections Reached` exception.

As near as I can tell, Home Assistant connection attempts each start a new websocket, which don't get properly closed.

This workaround involves using a firewall on the Klipper host to block connections to moonraker port 7125 when klipper is shutdown.

# Install

## Prerequisites
You will need UFW (Uncomplicated Firewall) installed and running on your klipper computer.
If you already have it configured, skip to the Install section. If not, follow these basic steps.

* Enable UFW: `sudo ufw enable`
* Allow SSH connections: `sudo ufw allow to any port 22`
* Allow http connections to Fluid/Mainsail: `sudo ufw allow to any port 80`
* If you run Fluid/Mainsail over HTTPS: `sudo ufw allow to any port 443`

For a more restrictive setup, you can set a `from` IP address or range in the above commands. I have this set to only allow connections from my local network. For example:

`sudo ufw allow from 192.168.1.0/24 to any port 80`

If your local subnet is not 192.168.1, you will need to change it in the above commands.

## Installation
Download this repository to the klipper computer:

`git clone https://github.com/akmjoe/hass-moonraker-firewall`

Change to the cloned directory:
`cd hass-moonraker-firewall`

If you have a different subnet than 192.168.1, open the file `moonraker_firewall` and change the following line:
`subnet="192.168.1.0/24` to the ip range you need.

Copy the files to where we need them:

```
sudo cp moonraker_firewall.service /etc/systemd/system/
sudo cp moonraker_firewall /usr/bin/moonraker_firewall
```

Change permissions so the script is executable:
`sudo chmod o+x /usr/bin/moonraker_firewall`

Reload systemd:
`sudo systemctl daemon-reload`

Start the service:
`sudo systemctl start moonraker_firewall`

This will only run until you reboot.
To enable running at startup:
`sudo systemctl enable moonraker_firewall`
## Check if it is working

First, check if the script is running:
`sudo systemctl status moonraker_firewall`

It should show as `active (running)`; if it is shutdown, make sure you set execute permission on the script.
You can test the script for errors by running `sudo /usr/bin/moonraker_firewall`.

Once the script is running, you can test its function. We need to check the firewall:
`sudo ufw status`

Look for the rule on port 7125. Start klippy and check it again (wait at least 10 seconds for the script to cycle).
Port 7125 should have a ALLOW rule for 192.168.1.0/24 when klippy is running, and DENY when shutdown.


## Stopping
To stop currently running service:
`sudo systemctl stop moonraker_firewall`

If the service is enabled at startup, it will restart next boot. To disable completely:
`sudo systemctl disable moonraker_firewall`

## Uninstall
To uninstall completely, run both commands in the [Stopping](#stopping)  section. Then run:

```
sudo rm /etc/systemd/system/moonraker_firewall.service
sudo rm /usr/bin/moonraker_firewallmoonraker_firewall
```

# Credits

## Authors

* **Joe Rothrock** - *Initial work* - [akmjoe](https://github.com/akmjoe)

## License

This project is licensed - see the [LICENSE.md](LICENSE.md) file for details