# Interfaces & IP Plan (sentry-gate01)

This file documents the NIC layout, interface naming, and IP plan for `sentry-gate01` — the firewall and rsyslog relay VM in Operation Iron Watch 03.

---

## NIC Layout

`sentry-gate01` uses a dual-NIC design:

| Interface | Zone | Type | IP Address | Description |
|-----------|------|------|-----------|-------------|
| `enp0s8` | LAN | DHCP | 192.168.0.4 | Uplink to LAN — management + Graylog relay destination |
| `enp0s9` | DMZ | Static | 10.10.10.2/24 | DMZ-facing interface — rsyslog relay ingestion point |

> Note: Interface names are VirtualBox-assigned. Always verify with `ip a` after deployment. Do not confuse with Safeguard Host interface names (`enp3s0`, `wlo1`) — those are the physical host's interfaces.

---

## Addressing

### LAN (enp0s8)
- Type: DHCP from home router (192.168.0.1)
- Assigned: 192.168.0.4
- Safeguard Host (LAN): management/bastion — SSH allowed from here only
- soc-core03 (Graylog): 192.168.0.6 — rsyslog relay destination

### DMZ (enp0s9)
- Type: Static
- Subnet: 10.10.10.0/24
- sentry-gate01 DMZ IP: **10.10.10.2**
- web-arm01 (Raspberry Pi): 10.10.10.10
- Safeguard Host DMZ gateway: 10.10.10.1 (enp3s0)

---

## Why Dual-NIC Only

The LAN provides:
- Management plane access (SSH from Safeguard Host only)
- Graylog relay destination — soc-core03 lives on LAN and receives logs via enp0s8

The DMZ is an isolated subnet for `web-arm01`. There is no third zone required — the relay model handles cross-zone log flow without opening additional routes.

---

## Netplan Configuration

Static IP for the DMZ interface is declared via Netplan:
```
/etc/netplan/01-sentry-gate01.yaml
```

Configuration is persistent across reboots. Full Netplan config documented in `03-hardening/`.

---

## Trust Model

| Zone | Interface | Trust Level | SSH Inbound |
|------|-----------|------------|-------------|
| LAN | enp0s8 | Trusted | From Safeguard Host only |
| DMZ | enp0s9 | Untrusted | Not permitted |

---

## Status

- ✅ Dual-NIC confirmed — enp0s8 (LAN 192.168.0.4) + enp0s9 (DMZ 10.10.10.2)
- ✅ Static IP on enp0s9 via Netplan — persistent across reboots
- ✅ Trust model enforced via UFW (see `firewall-policy.md`)
