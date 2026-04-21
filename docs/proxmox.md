# Proxmox VE Setup

## Cluster Overview

2-node Proxmox VE 8.3 cluster running on MINISFORUM MS-01 mini PCs connected via 10G SFP+ DAC.

| Node | IP | CPU | RAM | Storage | Status |
|------|----|-----|-----|---------|--------|
| pve-node1 | 10.0.0.11 | i9-12900H | 64GB DDR5 | 2TB NVMe | Secondary |
| pve-node2 | 10.0.0.12 | i9-12900H | 64GB DDR5 | 2TB NVMe | Primary |

---

## VMs

| VM ID | Name | Node | IP | CPU | RAM | Purpose |
|-------|------|------|----|-----|-----|---------|
| 100 | docker-host | pve-node2 | 10.20.0.10 | 8 cores | 16GB | All Docker services |

---

## GPU Passthrough (VFIO/IOMMU)

Intel Iris Xe GPU passed through to docker-host for hardware transcoding.

### Enable IOMMU in Proxmox

Edit `/etc/default/grub`:
```bash
GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on iommu=pt"
```

Update grub and reboot:
```bash
update-grub
reboot
```

### Add VFIO Modules

Edit `/etc/modules`:
```
vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd
```

### VM Hardware Config

In Proxmox UI: VM → Hardware → Add → PCI Device
- Select Intel Iris Xe GPU
- ✅ All Functions
- ✅ ROM-Bar
- ❌ Primary GPU

### Verify Inside VM
```bash
ls -la /dev/dri/
# card0, card1, renderD128 should appear
```

---

## VM Migration Notes

### Live Migration Blockers
- VMs with PCIe passthrough **cannot** be live migrated
- VMs with local CD/DVD ISO mounted cannot be migrated

### Migration Procedure for docker-host
1. Shutdown VM
2. Hardware → CD/DVD → No Media
3. Hardware → Remove hostpci0
4. Migrate → select target node
5. Start VM on new node
6. Hardware → Add → PCI Device (re-add GPU from new node)
7. Start VM

---

## Cluster Management

```bash
# Check cluster status
pvecm status

# List all VMs across cluster
qm list

# Start/stop a VM
qm start <VMID>
qm stop <VMID>

# Check node resources
pvesh get /nodes/pve-node1/status
```

---

## Storage

| Storage | Type | Used For |
|---------|------|----------|
| local-lvm | LVM-thin | VM disks |
| NFS (Synology) | NFS | Shared media, Docker data |

### NFS Mount on Proxmox Nodes
Added via Datacenter → Storage → Add → NFS
- Server: 10.0.0.13
- Export: /volume1/[share]

---

## Autostart VMs

Ensure critical VMs start automatically after node reboot:
VM → Options → Start at boot: **Yes**
