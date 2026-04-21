# Network Configuration

## Overview

7-VLAN network architecture built on OPNsense with Ubiquiti switching and WiFi. Direct fiber connection to Verizon Fios ONT (ISP router bypassed).

---

## Hardware

| Device | Model | Role |
|--------|-------|------|
| Router/Firewall | ZHIANN N100 running OPNsense 26.1.2 | Edge routing, firewall, DHCP, DNS |
| Switch | Ubiquiti Pro XG 24 PoE | Core switching, 10G uplinks |
| Access Point | Ubiquiti U7 Pro XG | WiFi 7, multi-SSID |

---

## OPNsense Configuration

### WAN
- **Connection**: Direct to Verizon Fios ONT
- **Type**: DHCP (ISP router bypassed)
- **Speed**: 1 Gbps symmetric

### LAN Interface
- **Interface**: igc1 (parent for all VLAN sub-interfaces)
- **Note**: Never delete — required for all VLAN trunking

### DHCP
- **Service**: Dnsmasq (not Kea — Kea failed silently)
- **Scope**: All 7 VLANs

### DNS
- **Primary**: AdGuard Home (10.20.0.10)
- **Fallback**: 8.8.8.8 (recommended — prevents outage if AdGuard goes down)

---

## VLAN Configuration

| VLAN ID | Name | Subnet | Gateway | DHCP Range | Purpose |
|---------|------|--------|---------|------------|---------|
| 10 | Management | 10.0.0.0/24 | 10.0.0.1 | .100-.250 | Proxmox, NAS, OPNsense |
| 20 | Trusted | 10.10.0.0/24 | 10.10.0.1 | .100-.250 | Personal laptops, phones |
| 30 | Servers | 10.20.0.0/24 | 10.20.0.1 | .100-.250 | VMs, Docker containers |
| 40 | Media | 10.30.0.0/24 | 10.30.0.1 | .100-.250 | TVs, streaming devices |
| 50 | IoT | 10.40.0.0/24 | 10.40.0.1 | .100-.250 | Smart home devices |
| 60 | Lab | 10.50.0.0/24 | 10.50.0.1 | .100-.250 | Testing, experiments |
| 99 | Guest | 10.99.0.0/24 | 10.99.0.1 | .100-.250 | Guest WiFi |

---

## Firewall Rules

**Important**: Rules must be created in "Rules [new]" interface — NOT legacy "Rules".

Each VLAN has a pass rule:
- Source: [VLAN net]
- Destination: any
- Protocol: IPv4

### WAN Rules
- Port 32400 → 10.20.0.10 (Plex remote access)
- Port 443/80 → 10.20.0.10 (Plex web)

---

## WiFi Configuration

| SSID | VLAN | Band | Purpose |
|------|------|------|---------|
| Valar Morghulis | Trusted (20) | WiFi 7 | Personal devices |
| A Wifi Has No Name | IoT (50) | WiFi 6 | Smart home |
| Valar Dohaeris | Guest (99) | WiFi 6 | Guests |

### UniFi Switch Port 5 (AP uplink)
- Native VLAN: Default
- Tagged VLANs: 10, 40, 99 (Allow All)

### UniFi Notes
- Default network (192.168.1.0/24) cannot be edited in third-party gateway mode
- AP management stays on Default/LAN VLAN
- WiFi client traffic uses tagged VLANs

---

## NAT

- **Mode**: Automatic outbound NAT
- **Coverage**: All VLANs (Guest, IoT, LAN, Lab, Media, Servers, Trusted, Loopback, 127.0.0.0/8)

---

## Port Forwarding

| External Port | Internal Host | Internal Port | Service |
|---------------|--------------|---------------|---------|
| 32400 | 10.20.0.10 | 32400 | Plex Media Server |

---

## Static IPs

| Host | IP | Role |
|------|----|------|
| OPNsense | 10.0.0.1 | Default gateway |
| pve-node1 | 10.0.0.11 | Proxmox node 1 |
| pve-node2 | 10.0.0.12 | Proxmox node 2 |
| Synology NAS | 10.0.0.13 | NAS / NFS server |
| docker-host | 10.20.0.10 | Docker VM |
