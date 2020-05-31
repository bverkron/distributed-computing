# Precursor / Disclaimer

When running a host for a distributed computing platform such as FluidStack or QBlocks on your machine you should assume there are zero steps taken by the provider to protect you or your system. Any such software / install should be assumed to be a security risk. As such you should ensure there is no sensitive / private information on your host machine and ideally have it totally isolated from your network (LAN blocked w/WAN only access).

# Intent

This is a side project to investigate the security aspects of being a host for a distributed computing platform and experiment with ways to improve it from the host perspective without interfering with the operation of it. We will focus primarily on Ubuntu and Docker as those the most common requirements at the time of this writing.

# Terms
* **Distributed Computing Host (aka host)**: The machine that you are using to host a [node](https://en.wikipedia.org/wiki/Node_(networking)#Distributed_systems) for the distributed computing platfor.

# Network Security

First and foremost you should make sure that your host is not able to communicate with other LAN devices, much like you would with [IoT devices](https://en.wikipedia.org/wiki/Internet_of_things). Such devices can pose security risks and allow attackers to access devices within your network that contain sensitive information.

## Scenario #1 - Single boot host
If you only have a single OS on your host system (the one used to run the distributed computing node) then you should block LAN access at the router level. This varies greatly by router manufacturer so check the docs for your router or other networking gear.

## Scenario #2 - Dual boot host

My host is currently dual boot Ubuntu / Windows 10. Sometimes I have a need to boot into Windows and don't want the LAN block in place thus decided to setup the block at the host OS level rather than the router level.

Many ways to accomplish this but I chose an `iptables` because it's fairly scriptable.

The core is these `iptables` commands.

**lan-block.sh**
```
iptables -I OUTPUT -m iprange --dst-range 192.168.1.2-192.168.1.254 -j REJECT
iptables -I INPUT -m iprange --src-range 192.168.1.2-192.168.1.254 -j REJECT
```

That won't persist across reboots so you'd have to use `iptables-persistent` or another method to make it persist.

I chose a custom service to do this on boot because I didn't like the way `iptables-persistent` and some other options use a full dump / restore method meaning you need to remember to re-run the dump portion when you modify iptables, etc. I wanted it to be additive so my custom rules just got added to whatever is already configured in iptables at any given time.

This should be loaded up before the network components of the OS start up do to the `WantedBy` line.

**lan-block.service**
```
[Unit]
Description=Apply LAN block iptables rules

[Service]
ExecStart=/usr/local/bin/lan-block.sh

[Install]
WantedBy=network-pre.target
```

Script to enable it all

**install-lan-block.sh**
```
cp lan-block.sh  /usr/local/bin/
chmod +x  /usr/local/bin/lan-block.sh
cp lan-block.service /etc/systemd/system
systemctl start lan-block.service
systemctl enable lan-block.service
```
