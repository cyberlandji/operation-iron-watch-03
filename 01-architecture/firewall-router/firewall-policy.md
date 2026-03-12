# Firewall Policy Baseline (FPB) — Default Deny

---

## Purpose

Establish a **"default deny"** baseline on `sentry-gate01` so that:
- Only explicitly allowed traffic is possible
- Management access is tightly controlled
- Log relay chain operates on clear, intentional flows

---

## Baseline Posture

UFW is enabled with:
- `"Default deny incoming"`
- `"Default deny outgoing"`
- `"Default deny routed (forwarded)"`

This means:
- `sentry-gate01` is not a transparent router
- Nothing crosses zones without an explicit allow rule

---

## Management Plane Rules

SSH to `sentry-gate01` is restricted to Safeguard Host only.

| Rule | Protocol | Source | Port | Action |
|------|----------|--------|------|--------|
| SSH management | TCP | Safeguard Host (LAN) | 22 | ALLOW |
| SSH (all others) | TCP | Any | 22 | DENY |

**Rationale:**
- Safeguard Host is the sole admin entrypoint (bastion model)
- Prevents opportunistic management access from LAN or DMZ devices
- Reduces attack surface on the management plane

---

## Forwarding / Relay Rules

The log relay chain is active. The following flows are explicitly permitted:

| Source | Destination | Port | Protocol | Purpose |
|--------|-------------|------|----------|---------|
| web-arm01 (10.10.10.10) | sentry-gate01 (10.10.10.2) | 514 | TCP | rsyslog — DMZ log ingestion |
| sentry-gate01 (192.168.0.4) | soc-core03 (192.168.0.6) | 514 | TCP | rsyslog — relay to Graylog |

> No other cross-zone forwarding is permitted. DMZ → LAN lateral movement is blocked by default-deny posture.

---

## Logging

rsyslog relay is operational — all log sources from `web-arm01` (auth.log, access.log, eve.json) transit through `sentry-gate01` to Graylog on `soc-core03`. Firewall decisions ar
