# Firewall / Router Layer (sentry-gate01)

This directory documents the **firewall/router enforcement layer** introduced in Operation Iron Watch 03 (IW03).

In IW03, `sentry-gate01` is not "just another VM" — it is the **network control point** used to:
- Enforce **segmentation** (LAN / DMZ / INTERNAL)
- Restrict **management access** (bastion model via Safeguard Host only)
- Provide a foundation for **defense-in-depth** (policy + telemetry + later inspection)

## Why it exists (IW02 lessons applied)
After IW02, we intentionally strengthened the architecture to avoid:
- Flat networks and implicit trust
- Uncontrolled east-west communication
- “Everything can talk to everything” management patterns

This firewall/router VM introduces:
- Explicit trust boundaries
- Default-deny posture
- A single management entrypoint

## Components (high level)
- **Host:** Safeguard Host (admin/bastion role; only allowed management source)
- **Firewall/Router VM:** `sentry-gate01` (Ubuntu Server 22.04 LTS)
- **Zones:** LAN (management), DMZ (public-facing service zone), INTERNAL (SIEM & core services zone)

## Implementation status (LOCKED MILESTONE)
✅ Multi-NIC design + static addressing for DMZ/INTERNAL  
✅ IP forwarding enabled (explicit routing capability)  
✅ UFW enabled with **default deny** (incoming / outgoing / routed)  
✅ SSH management restricted to **Safeguard Host only**  

🚧 Forwarding rules between zones will be added in later IW03 steps (controlled allow-lists).

## Files
- `network-design.md` — conceptual zone model & trust boundaries
- `interfaces.md` — NIC mapping + IP addressing scheme
- `routing.md` — routing philosophy and IP forwarding decision
- `firewall-policy.md` — baseline firewall posture and management plane rules

EOF
