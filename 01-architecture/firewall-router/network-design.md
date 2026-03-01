# Network Design (LAN / DMZ / INTERNAL)

## Goal
Introduce **network segmentation** and a **policy enforcement point** to support defense-in-depth in IW03.

This design separates:
- **Management traffic** (LAN)
- **Exposed services** (DMZ)
- **Security tooling / core systems** (INTERNAL)

## Zones (conceptual)
### 1) LAN (Management Zone)
- Purpose: administrative access and management plane
- Contains: Safeguard Host (admin workstation / bastion)
- Trust level: highest (but still restricted by policy)
- Access rule principle: *management is explicit, not assumed*

### 2) DMZ (Service Zone)
- Purpose: isolate public-facing workload(s)
- Contains: `web-arm01` (later moved here)
- Trust level: low (assume exposed / probed)
- Access rule principle: *DMZ is monitored and constrained*

### 3) INTERNAL (Core Zone)
- Purpose: protect detection stack and internal services
- Contains: `soc-core04` (later placed here)
- Trust level: medium/high (but still least-privileged)
- Access rule principle: *internal ≠ trusted-by-default*

## Trust boundaries (what we enforce)
- DMZ must not freely reach INTERNAL
- INTERNAL must not freely reach DMZ
- Only explicitly permitted flows are allowed
- Management access to `sentry-gate01` is restricted to **Safeguard Host**

## Why this is defense-in-depth
Even if one layer fails (e.g., a DMZ host is compromised):
- Segmentation limits blast radius
- Policies reduce reachable attack surface
- Monitoring/detection has clearer signal (zone-aware telemetry)

## IW03 narrative alignment
- IW03 = **implementation and architecture lab**
- IW04 = controlled offensive validation (Penetration Testing)
- IW05 = tuning + hardening based on results

EOF
