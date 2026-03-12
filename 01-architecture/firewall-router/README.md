# Firewall / Relay Layer (sentry-gate01)

This directory documents the **firewall and rsyslog relay layer** introduced in Operation Iron Watch 03 (IW03).

In IW03, `sentry-gate01` is not "just another VM" — it is the **network control point** that:
- Enforces segmentation between LAN (192.168.0.0/24) and DMZ (10.10.10.0/24)
- Relays logs from the DMZ to Graylog — the DMZ host has no direct SIEM access
- Restricts management access (bastion model via Safeguard Host only)
- Provides a foundation for **defense-in-depth** (policy + telemetry + log forwarding)

---

## Why It Exists (IW02 Lessons Applied)

After IW02, we intentionally strengthened the architecture to avoid:
- Flat networks and implicit trust
- Uncontrolled east-west communication
- Invisible log paths — the IW02 SSH compromise was undetected because auth.log never reached the SIEM

This layer introduces:
- Explicit trust boundaries between DMZ and LAN
- Default-deny posture via UFW
- A single management entrypoint
- A controlled log relay chain — web-arm01 → sentry-gate01 → Graylog

---

## Components

| Component | Details |
|-----------|---------|
| **Safeguard Host** | enp3s0: 10.10.10.1 (DMZ gateway), wlo1: LAN — admin/bastion role |
| **sentry-gate01** | Ubuntu Server 22.04 LTS — dual-NIC: 10.10.10.2 (DMZ) / 192.168.0.4 (LAN) |
| **TP-Link TL-SF1005D** | Unmanaged switch — physical DMZ segment hub |
| **web-arm01** | Raspberry Pi 3B+ — 10.10.10.10/24 — DMZ endpoint |
| **soc-core03** | Graylog 5.2 — 192.168.0.6 — log destination |
| **Zones** | LAN (192.168.0.0/24) — management/SIEM, DMZ (10.10.10.0/24) — public-facing |

---

## Log Relay Chain
