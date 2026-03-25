# HomeLab
Documenting the creation of my HomeLab
## 📐 Infrastructure Documentation

### Rack Layout
Professional 42U rack configuration with:
- OPNsense firewall + 7-VLAN segmentation
- Proxmox VE cluster (2 nodes, 10G cluster network)
- NAS with NFS storage
- UniFi network management
- Full documentation: [Rack Layout Diagram](./docs/Rack_Layout_Diagram_Public.docx)

### Network Architecture
- **VLAN 1:** Management (infrastructure)
- **VLAN 10:** Trusted (personal devices)
- **VLAN 20:** Servers (VMs/containers)
- **VLAN 30:** Media (Plex, Home Assistant)
- **VLAN 40:** IoT (smart home, restricted)
- **VLAN 50:** Lab (testing/development)
- **VLAN 99:** Guest (internet-only, isolated)
