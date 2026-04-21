# Docker Services

All services run on `docker-host` VM (`10.20.0.10`) — Ubuntu Server 24.04 LTS with Intel Iris Xe GPU passthrough.

## VM Configuration
- **CPU**: 8 cores (from Proxmox node)
- **RAM**: 16GB
- **Storage**: 500GB local NVMe + NFS mounts
- **GPU**: Intel Iris Xe (VFIO/IOMMU passthrough)
- **Network**: Host mode (direct access to VLAN 30 - Servers)

---

## Compose File Structure

```
~/docker-compose/
├── media-stack.yml      # Plex, Jellyfin, Radarr, Sonarr, Prowlarr, SABnzbd, qBittorrent, Overseerr, Tautulli
├── adguard.yml          # AdGuard Home
├── homepage.yml         # Homepage dashboard
└── dockhand.yml         # Container management
```

---

## Media Stack

### Plex Media Server
- **Image**: `plexinc/pms-docker:latest`
- **Port**: 32400
- **GPU**: Intel Iris Xe hardware transcoding (Quick Sync)
- **Storage**: `/mnt/synology/media` (NFS)
- **Config**: `~/docker-data/plex`
- **Note**: Requires `privileged: true` for GPU/libusb access

### Jellyfin
- **Image**: `jellyfin/jellyfin:latest`
- **Port**: 8096
- **Storage**: `/mnt/synology/media` (read-only NFS)
- **Config**: `~/docker-data/jellyfin`
- **Note**: Open-source Plex alternative, no cloud dependency

### Radarr
- **Image**: `lscr.io/linuxserver/radarr:latest`
- **Port**: 7878
- **Purpose**: Automated movie downloading and organization

### Sonarr
- **Image**: `lscr.io/linuxserver/sonarr:latest`
- **Port**: 8989
- **Purpose**: Automated TV show downloading and organization

### Prowlarr
- **Image**: `lscr.io/linuxserver/prowlarr:latest`
- **Port**: 9696
- **Purpose**: Indexer manager for Radarr/Sonarr (NZBGeek configured)

### SABnzbd
- **Image**: `lscr.io/linuxserver/sabnzbd:latest`
- **Port**: 8080
- **Purpose**: Usenet downloader
- **Performance**: 100 MB/s sustained (Newshosting, 30 connections)
- **Storage**: `/mnt/synology/downloads`

### qBittorrent
- **Image**: `lscr.io/linuxserver/qbittorrent:latest`
- **Port**: 8090
- **Storage**: `/mnt/synology/downloads`

### Overseerr
- **Image**: `sctx/overseerr:latest`
- **Port**: 5055
- **Purpose**: Media request management for family/friends

### Tautulli
- **Image**: `lscr.io/linuxserver/tautulli:latest`
- **Port**: 8181
- **Purpose**: Plex monitoring, statistics, and notifications

---

## Infrastructure Services

### AdGuard Home
- **Port**: 3030 (UI), 53 (DNS)
- **Purpose**: Network-wide DNS + ad blocking
- **Used by**: All VLANs via OPNsense DHCP DNS option
- **Critical**: DNS server for entire network — outage = no internet for all clients

### Immich
- **Image**: `ghcr.io/immich-app/immich-server:release`
- **Port**: 2283
- **Purpose**: Self-hosted Google Photos replacement
- **Features**: AI face recognition, object detection, hardware video transcoding (VAAPI)
- **Storage**: `/mnt/synology/photos`
- **Template**: `{{y}}/{{MM}}-{{MMM}}/{{filename}}`

### Homepage
- **Port**: 3000
- **Purpose**: Unified dashboard for all services

---

## NFS Mounts

```bash
# /etc/fstab entries
10.0.0.13:/volume1/media     /mnt/synology/media     nfs  defaults  0 0
10.0.0.13:/volume1/downloads /mnt/synology/downloads nfs  defaults  0 0
10.0.0.13:/volume1/photos    /mnt/synology/photos    nfs  defaults  0 0
```

---

## GPU Passthrough Setup

Intel Iris Xe GPU is passed through from Proxmox to docker-host VM using VFIO/IOMMU.

### Proxmox VM Hardware Config
```
hostpci0: [PCI_ADDRESS],pcie=1,rombar=1
```

### Docker Container Access
```yaml
devices:
  - /dev/dri:/dev/dri
privileged: true
```

### Verify GPU Access
```bash
ls -la /dev/dri/
# Should show: card0, card1, renderD128
```

---

## Useful Commands

```bash
# Check all running containers
docker ps

# View logs for a service
docker logs [service] --tail 50

# Restart entire media stack
cd ~/docker-compose
docker compose -f media-stack.yml up -d

# Check container resource usage
docker stats --no-stream

# Verify Plex is responding
curl -I http://localhost:32400/identity
```
