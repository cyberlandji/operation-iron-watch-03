# Interfaces & IP Plan (sentry-gate01)

This file documents the **NIC layout**, interface naming, and IP plan.

## NIC layout (LOCKED)
`sentry-gate01` uses a multi-NIC design:

- **LAN (Management):** DHCP from home router (CGNAT)
- **DMZ:** Static IP
- **INTERNAL:** Static IP

> Note: interface names are Linux/VirtualBox assigned (e.g., `enp0s8`, `enp0s9`, `enp0s10`).

## Actual addressing (current)
### LAN (Management)
- DHCP: `192.168.0.0/24`
- `sentry-gate01` LAN IP example: `192.168.0.84/24` (DHCP)
- Safeguard Host example: `192.168.0.7` (allowed to SSH)

### DMZ (Static)
- Subnet: `10.10.10.0/24`
- Gateway IP (on sentry-gate01): `10.10.10.1/24`

### INTERNAL (Static)
- Subnet: `10.20.20.0/24`
- Gateway IP (on sentry-gate01): `10.20.20.1/24`

## Why DHCP only on LAN
LAN is the upstream network (home router) and provides:
- IP assignment
- DNS
- Optional internet egress (when allowed)

DMZ and INTERNAL are intentionally deterministic and isolated:
- easier to reason about flows
- easier to write firewall rules
- prevents accidental exposure

## Netplan location
Static IPs are declared via netplan, e.g.:
- `/etc/netplan/00-installer-config.yaml`

(Exact config is documented in build evidence / notes during IW03 steps.)

EOF
