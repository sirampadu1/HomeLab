# OPNsense Configuration Notes

## Critical Notes

- **Firewall rules**: Always use "Rules [new]" interface, NOT legacy "Rules"
- **LAN interface (igc1)**: Parent for all VLAN sub-interfaces — NEVER delete
- **DHCP**: Use Dnsmasq, NOT Kea (Kea failed silently in testing)

---

## Gotchas & Lessons Learned

### Outbound NAT
- Mode: Automatic
- Covers all VLANs automatically
- If clients can't reach internet but router can → check NAT mode first

### WAN Gateway
- Type: Dynamic (DHCP from ISP)
- If gateway shows missing after reboot → Interfaces → WAN → Renew DHCP lease

### DNS Redundancy
Always configure a fallback DNS in System → Settings → General:
```
Primary DNS: 10.20.0.10  (AdGuard Home)
Fallback DNS: 8.8.8.8    (Google — for when AdGuard is down)
```

### Firewall Rule Order
Rules are evaluated top-to-bottom. Place more specific rules above general rules.

### VLAN Firewall Rules Template
Each VLAN needs minimum:
- **Action**: Pass
- **Interface**: [VLAN interface]
- **Source**: [VLAN] net  
- **Destination**: any
- **Protocol**: IPv4

---

## Port Forwarding (Destination NAT)

| Service | External Port | Internal IP | Internal Port |
|---------|--------------|-------------|---------------|
| Plex | 32400 | 10.20.0.10 | 32400 |

---

## Diagnostics Quick Reference

```
Diagnostics → Ping          # Test connectivity from OPNsense
Diagnostics → Packet Capture # Capture traffic on interface
Firewall → Log Files → Live View # Real-time firewall log
System → Gateways → Single  # Check gateway status
```
