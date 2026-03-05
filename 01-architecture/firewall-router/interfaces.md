# Interfaces & IP Plan (sentry-gate01)

This file documents the NIC layout, interface naming, and IP plan for `sentry-gate01` — the Firewall / Router VM in Operation Iron Watch 03.

---

## NIC Layout (LOCKED)

`sentry-gate01` uses a dual-NIC design:

| Interface | Zone | Type | Description |
|-----------|------|------|-------------|
| `wlo1` | LAN | DHCP (from home router) | Uplink to home router — internet access + LAN management |
| `enp3s0` | DMZ | Static | Dedicated interface facing the DMZ subnet |

> Note: Interface names are Linux/VirtualBox assigned (e.g., `enp3s0`, `wlo1`). These may vary depending on the host — always verify with `ip a` after deployment.

---

## Addressing (Current)

### LAN (wlo1)
- Type: DHCP from home router (192.168.0.1)
- Range: 192.168.0.0/24
- Safeguard Host example: 192.168.0.7 (allowed to SSH)
- soc-core04 (Graylog): 192.168.0.5

### DMZ (enp3s0)
- Type: Static
- Subnet: 10.10.10.0/24
- sentry-gate01 DMZ gateway IP: 10.10.10.x (to be confirmed on deployment)
- web-arm01 (Raspberry Pi): 10.10.10.10

---

## Why Dual-NIC Only

The LAN is the upstream network (home router) and provides:
- IP assignment (DHCP)
- Internet access
- Management plane access (SSH from Safeguard Host only)

The DMZ is a dedicated isolated subnet for public-facing services (`web-arm01`). There is no third zone — soc-core04 lives in the LAN and receives logs via the LAN interface of `sentry-gate01`.

---

## Netplan Location

Static IPs are declared via Netplan, e.g.:

```
/etc/netplan/01-sentry-gate01.yaml
```

Netplan configuration files will be documented in `03-hardening/` once deployment is complete.

---

## Trust Model Summary

| Zone | Trust Level | SSH Access |
|------|------------|------------|
| LAN (wlo1) | Trusted | From Safeguard Host only |
| DMZ (enp3s0) | Untrusted | Not permitted inbound |

> Management of `sentry-gate01` is only permitted from the LAN side, restricted to Safeguard Host (192.168.0.7).
