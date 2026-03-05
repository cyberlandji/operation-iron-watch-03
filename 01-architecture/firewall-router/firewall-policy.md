# Firewall Policy Baseline (FPB) — Default Deny

---

## Purpose

Establish a **"default deny"** baseline on `sentry-gate01` so that:
- Only explicitly allowed traffic is possible
- Management access is tightly controlled
- Future detection/monitoring operates on clear, intentional flows

---

## Baseline Posture (LOCKED)

UFW is enabled with:
- `"Default deny incoming"`
- `"Default deny outgoing"`
- `"Default deny routed (forwarded)"`

This means:
- `sentry-gate01` is not a transparent router
- Nothing crosses zones until allow rules exist

---

## Management Plane Rule (LOCKED)

SSH to `sentry-gate01` is restricted to Safeguard Host only.

**Example intent:**
- Allow TCP/22 from Safeguard Host IP (192.168.0.7)
- Deny SSH from all other sources (including DMZ)

**Rationale:**
- Safeguard Host is the bastion / admin entrypoint
- Reduces attack surface on management plane
- Prevents opportunistic admin access from other LAN devices

---

## Forwarding Rules (Not Yet Added)

At this stage:
- Routed traffic is denied by default
- DMZ → LAN forwarding is not allowed

**Planned allow-lists (next IW03 steps):**

| Source | Destination | Port | Purpose |
|--------|-------------|------|---------|
| web-arm01 (10.10.10.10) | sentry-gate01 | 514/TCP | rsyslog log forwarding |
| sentry-gate01 | soc-core04 (192.168.0.5) | 514/TCP or 12201/UDP | Graylog log ingestion |

> No other cross-zone forwarding will be permitted.

---

## Logging Intent (Future Enhancement)

To support detection engineering later in IW03:
- Enable visibility into all permitted connections (EVE logs and/or packet inspection)
- Correlate firewall decisions with SIEM telemetry in Graylog
