# Firewall / Router Layer (sentry-gate01)

This directory documents the **firewall/router enforcement layer** introduced in Operation Iron Watch 03 (IW03).

In IW03, `sentry-gate01` is not "just another VM" — it is the **network control point** used to:
- Enforce segmentation between LAN (192.168.0.0/24) and DMZ (10.10.10.0/24)
- Restrict management access (bastion model via Safeguard Host only)
- Provide a foundation for **defense-in-depth** (policy + telemetry + log forwarding)

---

## Why It Exists (IW02 Lessons Applied)

After IW02, we intentionally strengthened the architecture to avoid:
- Flat networks and implicit trust
- Uncontrolled east-west communication
- "Everything can talk to everything" management patterns

This firewall/router VM introduces:
- Explicit trust boundaries
- Default-deny posture
- A single management entrypoint

---

## Components (High Level)

| Component | Details |
|-----------|---------|
| **Host** | Safeguard Host (192.168.0.7) — admin/bastion role, only allowed management source |
| **Firewall/Router VM** | `sentry-gate01` — Ubuntu Server 22.04 LTS |
| **Zones** | LAN (management, 192.168.0.0/24), DMZ (public-facing service zone, 10.10.10.0/24) |

---

## Implementation Status (LOCKED MILESTONE)

- ✅ Multi-NIC design + static addressing for DMZ
- ✅ IP Forwarding enabled (explicit routing capability)
- ✅ UFW enabled with **default deny** (incoming / outgoing / routed)
- ✅ SSH management restricted to **Safeguard Host only**

> Forwarding rules between zones will be added in later IW03 steps (controlled allow-lists).

---

## Files

- `network-design.md` — conceptual zone model & trust boundaries
- `interfaces.md` — NIC mapping + IP addressing scheme
- `routing.md` — routing philosophy and IP forwarding decision
- `firewall-policy.md` — baseline firewall posture and management plane rules
