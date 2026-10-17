# Home Segmentation Network

## Overview

This project documents my home lab for practicing Linux networking, routing, DHCP, NAT, firewall rules, troubleshooting, and network segmentation.

The goal is to build the network step by step and relate the concepts to real-world firewall and network administration.

## Phase 1 — Ubuntu Router and Firewall

Phase 1 uses Ubuntu Server as a router/firewall between a trusted network and a separate Lab/User network.

### Lab Design

```text
Trusted / Upstream Network
10.10.10.0/24
        |
     Ubuntu Server
        |
Lab / User Network
10.20.20.0/24
        |
    Lab Client




The Ubuntu server uses separate network interfaces for the trusted/upstream side and the Lab network.
A third network interface is currently unused and available for future testing.

## Project Objective

The goal of Phase 1 was to build a separate Lab network using Ubuntu Server and understand how routing, DHCP, NAT, and firewall segmentation work together.

The Lab needed to:

- Receive IP addresses automatically.
- Use the Ubuntu server as its default gateway.
- Reach the Internet.
- Stay separated from the trusted/home network.
- Allow the trusted/admin side to reach the Lab.
- Keep working after a reboot.


## Technologies Used

- Ubuntu Server
- Netplan
- ISC DHCP Server
- IPv4 Forwarding
- nftables
- NAT / Masquerading
- SSH
- systemd
- journalctl
- Windows networking tools
- Git and GitHub

## Network Configuration

### Trusted / Upstream Side

- Interface: `enp7s0`
- Purpose: Home / trusted network connection
- Example subnet: `10.10.10.0/24`

### Lab Side

- Interface: `enp8s0`
- Purpose: Lab / user network
- Example gateway: `10.20.20.1/24`
- Example DHCP range: `10.20.20.100 - 10.20.20.200`

### Reserved Interface

- Interface: `enp2s0`
- Currently unused
- Available for future testing or another network segment

## Network Configuration

### Trusted / Upstream Side

- Interface: `enp7s0`
- Purpose: Home / trusted network connection
- Example subnet: `10.10.10.0/24`

### Lab Side

- Interface: `enp8s0`
- Purpose: Lab / user network
- Example gateway: `10.20.20.1/24`
- Example DHCP range: `10.20.20.100 - 10.20.20.200`

### Reserved Interface

- Interface: `enp2s0`
- Currently unused
- Available for future testing or another network segment

## DHCP, Routing, and NAT

### DHCP

The Ubuntu server provides IP addresses to devices connected to the Lab network.

Example Lab settings:

- Network: `10.20.20.0/24`
- Gateway: `10.20.20.1`
- DHCP range: `10.20.20.100 - 10.20.20.200`
- DNS: `8.8.8.8` and `1.1.1.1`

### Routing

IPv4 forwarding is enabled so Ubuntu can route traffic between the Lab network and the upstream network.

```text
net.ipv4.ip_forward = 1

NAT
NAT masquerading allows Lab devices to reach the Internet through the Ubuntu server.

Lab Client
    |
    v
Ubuntu Server
    |
    v
NAT / Masquerade
    |
    v
Upstream Network
    |
    v
Internet


Why this section matters: it shows that you understand these are **three different jobs**:

```text
DHCP   → gives the client network settings
Routing → moves traffic between networks
NAT     → translates Lab traffic toward the Internet

## Firewall Segmentation

nftables is used to control traffic between the trusted network and the Lab network.

Current policy:

```text
Lab → Internet        ALLOW
Lab → Trusted         DENY
Trusted → Lab         ALLOW
Established Traffic   ALLOW
Everything Else       DENY
```

The forwarding chain uses a default-deny policy:

```nft
policy drop;
```

Stateful connection tracking allows return traffic from connections that were already permitted:

```nft
ct state established,related accept
```

This is similar to a FortiGate firewall using explicit allow/deny policies with an implicit deny at the bottom.

### Simple FortiGate Comparison

| nftables | FortiGate |
|---|---|
| `accept` | ACCEPT policy |
| `drop` | DENY policy |
| `policy drop` | Implicit Deny |
| `ct state established,related` | Stateful session tracking |
| FORWARD chain | Traffic crossing the firewall |


## DHCP Troubleshooting and Fix

During testing, the Lab client sometimes received an APIPA address:

```text
169.254.x.x
```

This showed that the client was not receiving a valid DHCP lease.

### Root Cause

The DHCP service was starting while the Lab interface was not fully ready.

When `enp8s0` had no physical connection, its Lab IP was not available yet, so DHCP could not bind to the correct subnet.

The logs showed:

```text
No subnet declaration for enp8s0 (no IPv4 addresses)
```

### Fix

The following Netplan setting was added to the Lab interface:

```yaml
ignore-carrier: true
```

This keeps the Lab IP configured even when no Ethernet cable is connected.

After the change:

```text
Server boots
    |
    v
enp8s0 keeps its Lab IP
    |
    v
DHCP starts normally
    |
    v
Lab client connects later
    |
    v
Client receives a valid DHCP address
```

The fix was verified with a full reboot.

## Final Verification

After the configuration and troubleshooting were completed, Phase 1 was tested from both the server and Lab client.

### Server Checks

- Lab interface had the correct static IP.
- Default route used the upstream interface.
- DHCP service was active.
- IPv4 forwarding was enabled.
- nftables was active.
- Firewall and NAT rules were loaded correctly.

### Lab Client Checks

- Received a valid DHCP address.
- Reached the Ubuntu gateway.
- Reached the Internet by IP address.
- Resolved and reached Internet hostnames.
- Could not reach the trusted/home gateway.
- Could be reached from the trusted/admin side.

### Result

Phase 1 was successfully completed.

The Ubuntu server now works as a router, DHCP server, NAT gateway, and firewall between the trusted network and the Lab network.

## Technical Concepts Learned

- How DHCP assigns IP settings to clients.
- How a default gateway is used to leave a subnet.
- How IPv4 forwarding allows Linux to route traffic.
- How NAT/masquerading lets private Lab devices reach the Internet.
- How nftables controls traffic between networks.
- The difference between INPUT, FORWARD, and OUTPUT traffic.
- How stateful firewall rules allow return traffic.
- Why a separate subnet does not automatically mean secure segmentation.
- How APIPA addresses can point to a DHCP problem.
- How physical link state can affect interface configuration and services.
- How to verify networking step by step instead of changing multiple things at once.
- How Linux firewall concepts relate to FortiGate policies and session tracking.


## Future Plans

- Harden the Ubuntu server INPUT firewall rules.
- Restrict SSH management access to the trusted/admin network.
- Test exposed ports with tools such as Nmap.
- Review firewall and SSH logs.
- Improve SSH security and least-privilege access.
- Integrate the FortiGate 80F.
- Add VLANs for Management, Users, Lab, IoT, and Guest networks.
- Add a managed switch.
- Integrate the Cisco wireless access point.
- Add VPN access.
- Add monitoring and security visibility.
- Continue documenting each major project phase with Git and GitHub.

## Commands Practiced

These are some of the main commands used during Phase 1.

### Network Troubleshooting

```bash
ip -br addr
ip link
ip route
ip neigh
ping <IP-address>
```

- `ip -br addr` — quickly checks interfaces and IP addresses.
- `ip link` — checks interface and physical link state.
- `ip route` — displays the routing table and default gateway.
- `ip neigh` — displays nearby devices learned on the network.
- `ping` — tests network connectivity.

### Services and Logs

```bash
systemctl status <service>
systemctl is-active <service>
journalctl -u <service>
```

- `systemctl` — checks and manages Linux services.
- `journalctl` — reviews service logs for troubleshooting.

### Netplan

```bash
sudo netplan get
sudo netplan generate
sudo netplan apply
```

- `netplan get` — displays the current Netplan configuration.
- `netplan generate` — validates and generates the network configuration.
- `netplan apply` — applies network configuration changes.

### Routing

```bash
sysctl net.ipv4.ip_forward
```

Used to verify whether IPv4 forwarding is enabled.

### Firewall

```bash
sudo nft list ruleset
```

Displays the active nftables firewall and NAT rules.

### SSH

```bash
ssh username@server
```

Used to remotely manage the Ubuntu server from another computer.

### Git and GitHub

```bash
git init
git status
git add README.md
git commit -m "message"
git remote -v
git push
```

These commands were used to create the local Git repository, track changes, create commits, connect the project to GitHub, and upload the documentation.

