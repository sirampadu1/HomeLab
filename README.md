# 🏠 HomeLab Infrastructure

> A production-grade self-hosted media, storage, and networking infrastructure built on enterprise-class hardware — designed for high availability, performance, and security.

---

## 📋 Table of Contents
- [Overview](#overview)
- [Hardware](#hardware)
- [Network Architecture](#network-architecture)
- [Services](#services)
- [Key Achievements](#key-achievements)
- [Skills Demonstrated](#skills-demonstrated)
- [Phases](#phases)

---

## Overview

A fully self-hosted homelab running on a Proxmox VE cluster with 10GbE networking, 7-VLAN segmentation, hardware GPU transcoding, and a complete media automation pipeline. Designed with high availability in mind — critical services run on dedicated VMs with NFS-backed storage on a Synology NAS.

| Component | Details |
|-----------|---------|
| **Hypervisor** | Proxmox VE 8.3 (2-node cluster) |
| **Router/Firewall** | OPNsense 26.1 |
| **Switching** | Ubiquiti Pro XG 24 PoE (10G) |
| **WiFi** | Ubiquiti U7 Pro XG |
| **Storage** | Synology DS923+ (72TB usable, SHR-1) |
| **WAN** | 1 Gbps symmetric fiber (Verizon Fios) |
| **UPS** | APC UPS + PDU in 42U rack |

---

## Hardware

### Compute
**2x MINISFORUM MS-01**
- CPU: Intel Core i9-12900H
- RAM: 64GB DDR5
- Storage: 2TB NVMe
- GPU: Intel Iris Xe (hardware transcoding)
- Interconnect: 10G SFP+ DAC cable
- Running: Proxmox VE 8.3 cluster

| Node | Hostname | IP | Role |
|------|----------|----|------|
| Node 1 | pve-node1 | 10.0.0.11 | Proxmox cluster node |
| Node 2 | pve-node2 | 10.0.0.12 | Proxmox cluster node (primary) |

### Storage
**Synology DS923+**
- Capacity: 72TB usable (SHR-1 RAID)
- Connectivity: 10GbE NFS
- Performance: 109 MB/s sustained write
- IP: 10.0.0.13

### Network
- **Router**: OPNsense 26.1.2 on ZHIANN N100
- **Switch**: Ubiquiti Pro XG 24 PoE (10G uplinks)
- **AP**: Ubiquiti U7 Pro XG

---

## Network Architecture

### VLAN Design

| VLAN | Name | Subnet | Gateway | Purpose |
|------|------|--------|---------|---------|
| 10 | Management | 10.0.0.0/24 | 10.0.0.1 | Infrastructure devices |
| 20 | Trusted | 10.10.0.0/24 | 10.10.0.1 | Personal devices |
| 30 | Servers | 10.20.0.0/24 | 10.20.0.1 | VMs & containers |
| 40 | Media | 10.30.0.0/24 | 10.30.0.1 | Streaming devices |
| 50 | IoT | 10.40.0.0/24 | 10.40.0.1 | Smart home devices |
| 60 | Lab | 10.50.0.0/24 | 10.50.0.1 | Testing & experiments |
| 99 | Guest | 10.99.0.0/24 | 10.99.0.1 | Guest network |

### WiFi SSIDs
| SSID | VLAN | Purpose |
|------|------|---------|
| Valar Morghulis | Trusted (20) | Personal devices |
| A Wifi Has No Name | IoT (50) | Smart home devices |
| Valar Dohaeris | Guest (99) | Guest access |

### OPNsense Configuration
- WAN: Direct connection to Verizon Fios ONT (bypassed ISP router)
- DHCP: Dnsmasq across all 7 VLANs
- DNS: AdGuard Home (ad-blocking + DNS filtering)
- Firewall: Per-VLAN rules with inter-VLAN routing
- NAT: Automatic outbound NAT for all VLANs

---

## Services

All services run as Docker containers on a dedicated Ubuntu Server 24.04 VM (`docker-host`, `10.20.0.10`) with Intel Iris Xe GPU passthrough for hardware transcoding.

### Media Stack
| Service | Port | Purpose |
|---------|------|---------|
| Plex | 32400 | Primary media server (hardware transcoding) |
| Jellyfin | 8096 | Open-source media server (backup) |
| Radarr | 7878 | Movie automation |
| Sonarr | 8989 | TV show automation |
| Prowlarr | 9696 | Indexer management |
| SABnzbd | 8080 | Usenet downloader |
| qBittorrent | 8090 | Torrent client |
| Overseerr | 5055 | Media request management |
| Tautulli | 8181 | Plex monitoring & analytics |

### Infrastructure
| Service | Port | Purpose |
|---------|------|---------|
| AdGuard Home | 3030 | DNS server + ad blocking |
| Homepage | 3000 | Unified service dashboard |
| Immich | 2283 | Self-hosted Google Photos replacement |

### Docker VM Specs
- OS: Ubuntu Server 24.04 LTS
- CPU: 8 cores | RAM: 16GB
- Storage: 500GB local + NFS mounts
- GPU: Intel Iris Xe passthrough (VFIO/IOMMU)

### NFS Mounts
```
/mnt/synology/media    → Plex/Jellyfin library
/mnt/synology/downloads → SABnzbd/qBittorrent destination
/mnt/synology/photos   → Immich storage
```

---

## Key Achievements

### Performance
- ✅ **109 MB/s** sustained NFS write to Synology over 10GbE
- ✅ **100 MB/s** Usenet download speed (Newshosting, 30 connections)
- ✅ **10-15+ concurrent** 1080p/4K Plex streams with <20% CPU
- ✅ **International remote streaming** confirmed working (family in Brazil)

### Technical
- ✅ Intel Iris Xe GPU passthrough (VFIO/IOMMU) for hardware transcoding
- ✅ 7-VLAN network segmentation with full inter-VLAN routing
- ✅ Complete media automation pipeline (request → download → organize → stream)
- ✅ Bypassed ISP router with direct ONT connection
- ✅ Proxmox 2-node HA cluster with live VM migration
- ✅ Self-hosted photo management with AI face recognition (Immich)

---

## Skills Demonstrated

| Category | Technologies |
|----------|-------------|
| **Virtualization** | Proxmox VE, KVM/QEMU, VM clustering, live migration |
| **Networking** | OPNsense, VLAN segmentation, firewall rules, NAT, DHCP, DNS |
| **Linux** | Ubuntu Server, Docker, NFS, IOMMU/VFIO, systemd |
| **Storage** | Synology NAS, SHR-1 RAID, NFS over 10GbE |
| **Security** | Network segmentation, firewall policies, DNS filtering, IoT isolation |
| **Hardware** | GPU passthrough, PCIe/IOMMU, 10GbE networking, UPS/PDU |
| **Monitoring** | Tautulli, AdGuard Home stats, Proxmox monitoring |
| **Automation** | Media pipeline (Radarr/Sonarr/Prowlarr/SABnzbd), Overseerr |

---

## Phases

| Phase | Status | Description |
|-------|--------|-------------|
| Phase 1 | ✅ Complete | Proxmox cluster foundation |
| Phase 2 | ✅ Complete | Network architecture (OPNsense + VLANs) |
| Phase 3 | ✅ Complete | Docker host + media stack |
| Phase 4 | ✅ Complete | GPU passthrough + hardware transcoding |
| Phase 5 | ✅ Complete | NAS integration + 10GbE storage |
| Phase 6 | 🔄 In Progress | High availability + redundancy improvements |

---

## Docs

- [Network Configuration](docs/network.md)
- [Proxmox Setup](docs/proxmox.md)
- [Docker Services](docs/docker-services.md)
- [Troubleshooting Log](docs/troubleshooting.md)
- [OPNsense Config Notes](configs/opnsense/README.md)

---

*Built and maintained by Stephen Gyamfi Ampadu*
