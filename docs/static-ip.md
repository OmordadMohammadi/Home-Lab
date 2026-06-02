Static IP Configuration — Ubuntu Server 24.04

What Is This?
By default Ubuntu Server gets its IP address automatically from the router via DHCP.
That means the address can change after a reboot or router restart.

A static IP fixes the server to one permanent address so you can always reach it
at the same location on the network — essential for SSH access and running services.

Tool Used: Netplan

Ubuntu Server 24.04 manages network configuration through Netplan. Settings are written in a YAML file and applied with a single command.

Steps 
1. Find your network interface name, in bash write
ip a

Look for your ethernet interface — typically named enp3s0, ens33 or eth0.
Note the name — you will need it in the config file.

3. Find your current gateway (router IP)
in bash write this command below

ip route show

Look for the line starting with default via — that IP is your gateway.

3. Edit the Netplan config file in bash.

sudo nano /etc/netplan/00-installer-config.yaml

4. Apply the configuration.
sudo chmod 600 /etc/netplan/00-installer-config.yaml
sudo netplan apply

5. Verify in bash.
ip a show enp3s0
ping -c 4 google.com

Config File Reference
See [configs/netplan-static-ip.yam](/configs/netplan-static-ip.yaml) for the full configuration with comments.

Problems Encountered

Wrong subnet in config initially entered `192.168.1.x` instead of `192.168.0.x` — the wrong network entirely.
Would have dropped all connectivity if applied.

Fix: Always verify your current IP with `ip a` and gateway with `ip route show`
before editing the Netplan file. Use those exact values.

File permissions warning
Running `sudo netplan apply` showed:

`WARNING: Permissions for /etc/netplan/00-installer-config.yaml are too open.`

Fix: `sudo chmod 600 /etc/netplan/00-installer-config.yaml`

Verification Commands

| Command | What It Shows |
|---------|--------------|
| `ip a` | All interfaces and their IP addresses |
| `ip route show` | Routing table including default gateway |
| `ping -c 4 google.com` | Confirms internet connectivity |
| `resolvectl status` | Confirms DNS servers are applied |

Result:
Server assigned permanent static IP on local network.
Reachable at the same address after every reboot. ✅
