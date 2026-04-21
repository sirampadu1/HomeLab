# Troubleshooting Log

A running log of issues encountered and resolved. Useful for reference and demonstrates real-world problem solving.

---

## 2026-04-16 — Complete Network Outage (No Internet)

### Symptoms
- All client devices lost internet access across all VLANs
- OPNsense WAN showed IP but no gateway
- WiFi and wired clients both affected

### Diagnosis Process
1. Confirmed ONT working (direct laptop connection to ONT worked)
2. OPNsense dashboard showed WAN_DHCP gateway active (71.166.43.1) — gateway not the issue
3. Ping to 8.8.8.8 from OPNsense succeeded — router had internet
4. Ping by IP from client worked, ping by name failed → **DNS issue**
5. AdGuard Home at 10.20.0.10 unreachable → **docker-host unreachable**
6. Proxmox node 1 found to be down → **root cause identified**

### Root Cause
Proxmox node 1 went down unexpectedly → docker-host VM went offline → AdGuard Home DNS server stopped → all clients lost DNS resolution → appeared as complete internet outage.

### Resolution
- Rebooted Proxmox node 1
- docker-host VM auto-started
- AdGuard Home restored
- DNS resolution restored across all VLANs

### Prevention
- Add `8.8.8.8` as fallback DNS in OPNsense (System → Settings → General)
- Consider second AdGuard instance on node 2 for DNS redundancy
- Investigate why node 1 went down (check Proxmox syslog)

---

## 2026-04-16 — Plex Container Crash Loop (libusb_init failed)

### Symptoms
```
Critical: libusb_init failed
Stopping Plex Media Server.
```
Plex container crash-looping after node 1 ungraceful shutdown.

### Diagnosis
- `docker logs plex --tail 100` showed repeating libusb_init failed crash
- `/dev/dri/` showed card0, card1, renderD128 present — GPU devices existed
- Issue was container lacking privileged access to USB/GPU subsystem after ungraceful shutdown

### Resolution
Added `privileged: true` to Plex service in `media-stack.yml`:

```yaml
plex:
  image: plexinc/pms-docker:latest
  container_name: plex
  network_mode: host
  privileged: true    # Added to fix libusb_init failed
  devices:
    - /dev/dri:/dev/dri
```

Then recreated container:
```bash
docker compose -f ~/docker-compose/media-stack.yml up -d plex
```

### Verification
```bash
curl -I http://localhost:32400/identity
# HTTP/1.1 200 OK
docker stats plex --no-stream
# CPU: ~1%, Memory: ~88MB
```

---

## 2026-04-16 — docker-host VM Migration (Node 1 → Node 2)

### Reason
Node 1 instability (SSH broken, services down after ungraceful shutdown). Migrated docker-host to node 2 for better resilience.

### Blockers Encountered
1. **Cannot migrate VM with local CD/DVD** — ISO was still mounted
2. **Cannot migrate VM with local resources: hostpci0** — GPU passthrough active

### Resolution
1. Shutdown docker-host VM
2. Hardware → CD/DVD → Set to "No Media"
3. Hardware → Remove hostpci0 (GPU)
4. Migrated VM to node 2 (offline migration, ~100GB disk transfer)
5. Re-added PCI device (Intel Iris Xe) from node 2's PCI list
6. Started VM on node 2
7. Verified all containers running: `docker ps`

### Result
- All services operational on node 2
- GPU passthrough working (no libusb errors)
- Plex hardware transcoding confirmed working

---

## Key Lessons Learned

1. **DNS is critical infrastructure** — a single DNS server is a single point of failure for the entire network. Always have a fallback.
2. **Ungraceful shutdowns cause container issues** — privileged mode or proper device remapping may be needed after power loss.
3. **PCIe passthrough blocks live migration** — plan for maintenance windows when GPU passthrough VMs need to move.
4. **"No internet" ≠ WAN is down** — always check DNS resolution separately from connectivity.
5. **Plex outages are not always local** — check status.plex.tv before deep-diving into local infrastructure.
