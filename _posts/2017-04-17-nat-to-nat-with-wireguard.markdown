---
layout: post
title:  "NAT-to-NAT VPN with WireGuard"
date:   2017-04-17 13:14:39
categories: vpn infrastructure research
---

A recent research project/idea required me to look into setting up a NAT-to-NAT VPN. The basic idea being that two NATed networks are able to communicate through a VPN and share resources. While researching possible VPN solutions, I remembered reading about [WireGuard](https://wireguard.io) a new VPN that aims to be fast, secure and lightweight. This seemed like the perfect opportunity to both try out a new VPN implementation and accomplish the goals of the research project.

## Planning

The basic idea was to connect to NATed environments, this meant neither of the environments had a [static] public IP address, and I required an intermediate server to act as a gateway.

I further required resource sharing to be limited, where NAT-A was able to access resources in NAT-B but NAT-B wasn't able to access the resources hosted in NAT-A. Setup also needed to be fast, lightweight and possible through a service script.

This resulted in the basic design was as follows:
![Infrastructure design](/assets/wireguard_vpn_setup.png)

For the VPN I used a private network range, which is usually unassigned, while both NATed networks had their own internal network ranges.

* VPN: 5.5.5.0/24
* NAT-A: 192.168.1.0/24
* NAT-B: 10.4.0.0/24

NAT-A needed a route for all traffic destined to 10.4.0.0/24 to be set to send traffic through the VPN, while NAT-B could not access the NAT-A network range.

## WireGuard Setup

WireGuard proved simple to setup in all my test environments. The intermediate/gateway server was an Ubuntu 16.04 server hosted in [DigitalOcean](https://digitalocean.com). While NAT-A was my local Fedora 25 host and the NAT-B host was an [Ubuntu 16.04 Cloud Image](https://cloud-images.ubuntu.com/xenial/) (I wanted to have cloud-init support). I'm not going to go into the installation of WireGuard here, the details are available on the [offical site](https://www.wireguard.io/install/).

### VPN Setup

The first component that needed to be configured was the actual VPN and getting NAT-A and NAT-B to communicate.

I'll use the following VARIABLES in the commands below, simply replace them with the correct values:

* SERVERPUB : ```cat publickey``` on the VPN server after using ```wg genkey```
* NATAPUB : ```cat publickey``` on NAT-A host
* NATBPUB : ```cat publickey``` on NAT-B host

#### Gateway server setup:

Ensure IP forwarding is enabled:

```
sysctl -w net.ipv4.ip_forward=1
```

And setup the VPN:
```
wg genkey | tee privatekey | wg pubkey > publickey
ip link add dev wg0 type wireguard
ip address add dev wg0 5.5.5.1/24
wg set wg0 private-key privatekey listen-port 12000
ip link set up dev wg0
```

#### NAT-A VPN setup:

```
wg genkey | tee privatekey | wg pubkey > publickey
ip link add dev wg0 type wireguard
ip address add dev wg0 5.5.5.3/24
wg set wg0 private-key privatekey peer SERVERPUB allowed-ips 5.5.5.0/24 endpoint vpn.server.com:12000 persistent-keepalive 10
ip link set up dev wg0
```

#### NAT-B VPN setup:

```
wg genkey | tee privatekey | wg pubkey > publickey
ip link add dev wg0 type wireguard
ip address add dev wg0 5.5.5.2/24
wg set wg0 private-key privatekey peer SERVERPUB allowed-ips 5.5.5.0/24 endpoint vpn.server.com:12000 persistent-keepalive 10
ip link set up dev wg0
```

This setups up our VPN and allows the NAT-A and NAT-B hosts to communicate through the 5.5.5.0/24 network. At this point it's not possible to share resources.

### Share Resources

Now that the two NATs are able to communicate, it's time to setup the sharing of resources. Remember that NAT-A should access NAT-B resources, but not the other way around.

To allow forwarding of the traffic by our gateway server, a few changes need to be made. Firstly the new Peering information needs to be enabled in WireGuard, so that WireGuard knows to tag and encrypt the correct values.

```
wg set wg0 peer NATAPUB allowed-ips 5.5.5.3/32,0.0.0.0/24
wg set wg0 peer NATBPUB allowed-ips 5.5.5.2/32,10.4.0.0/24
```

Some **iptables** foo needs to be applied to allow forwarding of traffic through the gateway. A new route is added for our NAT-B range that needs to be accessible.

```
ip route add 10.4.0.0/24 dev wg0

iptables -A FORWARD -i wg0 -j ACCEPT
iptables -A FORWARD -o wg0 -j ACCEPT
iptables -t nat -A POSTROUTING -o wg0 -j MASQUERADE
```

Next NAT-A needs to be setup to correctly route traffic for the resources in NAT-B.

For this, the WireGuard peer information is updated and a new route is added.

```
wg set wg0 private-key privatekey peer SERVERPUB allowed-ips 0.0.0.0/0 endpoint vpn.server.com:12000 persistent-keepalive 25
ip route add 10.4.0.0/24 via 5.5.5.1 dev wg0
```

Finally, the NAT-B host needs to be updated to be able to forward traffic to the resources inside it's LAN.

```
sysctl -w net.ipv4.ip_forward=1
iptables -A FORWARD -i wg0 -j ACCEPT
iptables -A FORWARD -o wg0 -j ACCEPT
iptables -t nat -A POSTROUTING -o wg0 -j MASQUERADE
iptables -t nat -A POSTROUTING -o ens3 -j MASQUERADE
```

At this point it should be possible for the NAT-A host to access resources inside NAT-B. Any requests for addresses in the 10.4.0.0/24 range will be routed through the VPN, forwarded by the gateway server to NAT-B, which will then forward to hosts inside the LAN.

### Extras

For my tests I needed to be able to make certain ports from the NAT-A host accessible to hosts within NAT-B. More specifically, I needed host 10.4.0.10 to spawn a reverse shell, without knowing the IP address of NAT-A. For this, I forwared a port range on the NAT-B host to our NAT-A host;

```
iptables -t nat -A PREROUTING -i ens3 -p tcp --dport 4000:5000 -j DNAT --to 5.5.5.3:4000-5000
```

#### As a service

Turning this into a service on NAT-B so that the VPN would come up automatically at boot was also needed and straight forward to implement. The iptables rules were saved and restored using iptables-save.

For the VPN service a systemd unit file was created:

```
[Unit]
Description=Starts the wireguard VPN
After=network-online.target

[Service]
ExecStart=/opt/wireguard/vpnup

[Install]
WantedBy=default.target
```

And the vpnup script in ```/opt/wireguard``` looked as follows:

```
#!/bin/sh -

SERVERPUB="$(cat /opt/wireguard/serverpub)"
SERVER="$(cat /opt/wireguard/serverhostname)"
PRIVATEKEY="/opt/wireguard/privatekey"

ip link del dev wg0
ip link add dev wg0 type wireguard
ip address add dev wg0 5.5.5.2/24
wg set wg0 private-key $PRIVATEKEY peer $SERVERPUB allowed-ips 5.5.5.0/24,10.4.0.0/24 endpoint $SERVER persistent-keepalive 10
ip link set up dev wg0
```
