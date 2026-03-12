# Network Design (LAN / DMZ)

This file documents the network segmentation model and policy enforcement point introduced in Operation Iron Watch 03 (IW03).

---

## Goal

Introduce **network segmentation** and a **policy enforcement point** to support defense-in-depth in IW03.

This design separates:
- Management traffic (LAN)
- Exposed services (DMZ)

---

## Zones

### Zone 1 — LAN (Management Zone)
- **Subnet:** 192.168.0.0/24
- **Purpose:** Administrative access, management plane, and internal services
- **Contains:** Safeguard Host (LAN via wlo1), soc-core03 / Graylog (192.168.0.6), sentry-gate01 LAN interface (192.168.0.4), home router (192.168.0.1)
- **Trust level:** High — SSH access restricted to Safeguard Host only
- **Access rule principle:** Management is explicit, not assumed

### Zone 2 — DMZ (Service Zone)
- **Subnet:** 10.10.10.0/24
- **Purpose:** Isolate public-facing workloads from the management plane
- **Contains:** web-arm01 / Raspberry Pi (10.10.10.10), sentry-gate01 DMZ interface (10.10.10.2), Safeguard Host DMZ gateway (10.10.10.1 via enp3s0)
- **Physical isolation:** TP-Link TL-SF1005D unmanaged switch
- **Trust level:** Low — assume exposed and probed at all times
- **Access rule principle:** DMZ is monitored and constrained — no free communication into LAN

---

## Trust Boundaries

- DMZ must not freely reach LAN
- LAN must not freely reach DMZ
- Only explicitly permitted flows are allowed:

| Source | Destination | Port | Purpose |
|--------|-------------|------|---------|
| web-arm01 (10.10.10.10) | sentry-gate01 (10.10.10.2) | 514/TCP | rsyslog log forwarding |
| sentry-gate01 (192.168.0.4) | soc-core03 (192.168.0.6) | 514/TCP | rsyslog relay to Graylog |

> No other DMZ → LAN communication is permitted.

---

## Why This Is Defense-in-Depth

Even if one layer fails (e.g. a DMZ compromise):
- Segmentation limits blast radius — attacker is contained in 10.10.10.0/24
- Default-deny UFW posture reduces available attack paths
- Physical switch isolation means DMZ traffic cannot bypass sentry-gate01
- Full log visibility (Suricata EVE JSON, auth.log, access.log) ensures detection even if prevention fails

---

## IW03 Narrative Alignment

| Episode | Focus |
|---------|-------|
| IW01 | Snort IDS, basic network visibility — no SIEM |
| IW02 | Graylog SIEM introduced, flat network — SSH compromise invisible |
| **IW03** | **Segmentation + hardening + detection engineering — direct response to IW02 gaps** |

> `web-arm01` was directly on the LAN in IW02, which allowed the SSH compromise to go undetected. The DMZ did not exist. IW03 closes that gap architecturally and at the detection layer.

---

## Status

- ✅ Zone segmentation implemented — LAN (192.168.0.0/24) / DMZ (10.10.10.0/24)
- ✅ Physical DMZ isolation via TP-Link TL-SF1005D switch
- ✅ sentry-gate01 dual-NIC confirmed and operational
- ✅ Trust boundaries enforced via UFW default-deny
- ✅ Log relay chain validated end-to-end
