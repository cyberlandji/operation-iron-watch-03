# Context — Why IW03 Exists

This document explains the context behind Operation Iron Watch 03: what Iron Watch 02 revealed, what decisions were made as a result, and what threat model IW03 is designed around.

---

## IW02 — What Happened

Operation Iron Watch 02 introduced Graylog as a SIEM and successfully detected web enumeration activity via HTTP 404 spike rules against Apache access logs on `web-arm01`.

However, during IW02 a **real SSH compromise occurred and was completely invisible on the SIEM.**

### Why It Was Invisible

| Gap | Root Cause |
|-----|-----------|
| `auth.log` was never ingested into Graylog | Log pipeline was incomplete — only `access.log` was forwarded |
| No network segmentation existed | `web-arm01` sat directly on the LAN (192.168.0.0/24) alongside all other hosts |
| No IDS on the web server | No tool was watching network-level traffic hitting `web-arm01` |
| Flat network trust model | Once inside `web-arm01`, an attacker could potentially reach LAN hosts freely |

### What the Attacker Could Do (Undetected)

- SSH into `web-arm01` with valid credentials (weak password)
- Move laterally toward LAN hosts with no network boundary in the way
- Persist on the host with no alert generated in Graylog
- Enumerate the internal network silently

> IW02 proved that a SIEM is only as good as what it ingests. Visibility gaps are security gaps.

---

## IW03 — The Direct Response

IW03 was designed specifically to close the gaps IW02 exposed. Every architectural decision in IW03 maps directly to a failure observed in IW02.

| IW02 Failure | IW03 Response |
|-------------|--------------|
| `auth.log` not ingested | `auth.log` ingested via rsyslog relay through `sentry-gate01` |
| No network segmentation | DMZ created — `web-arm01` isolated in 10.10.10.0/24 |
| No IDS on web server | Suricata deployed on `web-arm01` — EVE JSON forwarded to Graylog |
| Flat LAN trust model | `sentry-gate01` enforces default-deny between zones |
| SSH accessible with weak credentials | SSH key-only authentication enforced on all hosts |

---

## Threat Model (IW03 Scope)

IW03 is a **hardening and detection engineering** operation. No active attacker is simulated in this phase — that is IW04's role.

### Assumed Threat Profile

- **Entry point:** `web-arm01` (public-facing, exposed Apache2 web server in DMZ)
- **Attack vectors in scope for detection:**
  - Web enumeration / HTTP flood (L7)
  - SYN flood against the web server (L4)
  - ICMP flood (L3)
  - SSH brute-force or unauthorized access attempts (auth.log)
- **Lateral movement:** Addressed architecturally via DMZ segmentation — not simulated until IW04

### What IW03 Does NOT Cover

- Active exploitation or post-compromise activity — this is IW04
- Privilege escalation detection — this is IW04
- Red team simulation — this is IW04

---

## Design Decisions

### Decision 1 — DMZ via Dedicated Subnet
`web-arm01` moved from the LAN to an isolated DMZ subnet (10.10.10.0/24). This limits blast radius if `web-arm01` is compromised — the attacker cannot freely reach LAN hosts.

### Decision 2 — sentry-gate01 as Firewall + Log Relay
Rather than having Graylog pull logs directly from `web-arm01` (which would require opening a port from DMZ into LAN), `sentry-gate01` acts as both the network enforcement point and the rsyslog relay. Logs flow:

```
web-arm01 → sentry-gate01 → soc-core04 (Graylog)
```

This keeps the DMZ→LAN boundary intact and gives `sentry-gate01` a dual role that is well-documented and auditable.

### Decision 3 — Suricata over Snort
Suricata was chosen over Snort for `web-arm01` (Raspberry Pi, ARM architecture) because:
- Suricata is available natively on ARM via `apt`
- Outputs EVE JSON natively — cleaner ingestion into Graylog
- Multi-threaded — better performance on Pi hardware
- More commonly deployed in real SOC environments

### Decision 4 — SSH Key-Only Authentication
All hosts enforce SSH key-only authentication. Password-based SSH was the direct cause of the IW02 compromise. This is a non-negotiable hardening baseline going forward.

---

## IW03 Position in the Iron Watch Series

| Episode | Focus | Key Outcome |
|---------|-------|------------|
| IW01 | Snort IDS, basic network visibility | Proof of concept — detection without SIEM |
| IW02 | Graylog SIEM introduced | Real SSH compromise invisible — exposed critical gaps |
| **IW03** | **DMZ, hardening, detection engineering** | **Closes IW02 gaps — builds defensible architecture** |
| IW04 | Attack validation with Kali Linux | Validates IW03 hardening against real recon and initial access |

> IW03 is the bridge between "detection that failed" and "detection that works."
