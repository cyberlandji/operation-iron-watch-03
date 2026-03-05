# Network Design (LAN / DMZ)

This file documents the network segmentation model and policy enforcement point introduced in Operation Iron Watch 03 (IW03).

---

## Goal

Introduce **network segmentation** and a **policy enforcement point** to support defense-in-depth in IW03.

This design separates:
- Management traffic (LAN)
- Exposed services (DMZ)

---

## Zones (Conceptual)

### Zone 1 — LAN (Management Zone)
- **Subnet:** 192.168.0.0/24
- **Purpose:** Administrative access, management plane, and internal services
- **Contains:** Safeguard Host (192.168.0.7), soc-core04 / Graylog (192.168.0.5), home router (192.168.0.1)
- **Trust level:** High (restricted by policy — SSH from Safeguard Host only)
- **Access rule principle:** Management is explicit, not assumed

### Zone 2 — DMZ (Service Zone)
- **Subnet:** 10.10.10.0/24
- **Purpose:** Isolate public-facing workloads from the management plane
- **Contains:** web-arm01 / Raspberry Pi (10.10.10.10) — Apache2 + Suricata
- **Trust level:** Low (assume exposed / probed)
- **Access rule principle:** DMZ is monitored and constrained — no free communication into LAN

---

## Trust Boundaries (What We Enforce)

- DMZ must not freely reach LAN
- LAN must not freely reach DMZ
- Only explicitly permitted flows are allowed:
  - `web-arm01` → `sentry-gate01` → `soc-core04` for log forwarding (rsyslog, port 514)
  - No direct DMZ → LAN communication permitted outside this flow
- Management access to `sentry-gate01` is restricted to Safeguard Host only

---

## Why This Is Defense-in-Depth

Even if one layer fails (e.g. a DMZ compromise):
- Segmentation limits blast radius
- Policies reduce available attack surface
- Monitoring and log ingestion provides detection capability (Suricata EVE JSON, auth.log, access.log)

---

## IW03 Narrative Alignment

- IW01 = visibility (Snort, basic network monitoring)
- IW02 = SIEM introduced, but flat network — SSH compromise invisible
- **IW03 = segmentation + hardening + detection engineering — direct response to IW02 gaps**

> The DMZ did not exist before IW03. `web-arm01` was directly on the LAN in IW02, which allowed the SSH compromise to go undetected. This design closes that gap.
