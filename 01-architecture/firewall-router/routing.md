# Routing (IP Forwarding) — Philosophy & Controls

This file documents the routing philosophy and IP forwarding decisions for `sentry-gate01` in Operation Iron Watch 03.

---

## Why Routing Matters Here

Without routing, isolated zones cannot communicate even when needed. With routing, traffic can traverse zones — but must be controlled.

---

## Explicit Routing Principle (LOCKED)

Routing is treated as a **capability**, not a default behavior:
- We enable the kernel to forward packets **only when we intentionally decide**
- We pair routing with firewall policy to ensure **least privilege**

---

## Implementation Decision

IP forwarding was enabled intentionally for IW03. Persisted in `/etc/sysctl.conf`:

```bash
net.ipv4.ip_forward = 1
```

This means `sentry-gate01` is not a transparent router — nothing crosses zones until allow rules exist.

---

## Security Stance

Even with IP forwarding enabled:
- Default deny firewall posture remains the main enforcement layer
- Forwarding between DMZ and LAN is **not permitted until explicit allow rules are added**

---

## Permitted Flows (Current)

| Source | Destination | Port | Purpose |
|--------|-------------|------|---------|
| web-arm01 (10.10.10.10) | sentry-gate01 | 514/TCP | rsyslog log forwarding |
| sentry-gate01 | soc-core04 (192.168.0.5) | 514/TCP or 12201/UDP | Graylog log ingestion |
| Safeguard Host (192.168.0.7) | sentry-gate01 | 22/TCP | SSH management (LAN only) |

> All other cross-zone traffic is denied by default. DMZ → LAN free routing is not permitted.

---

## What Comes Next (IW04)

Routing alone is not enough. IW04 (attack validation) can show:
- Whether segmentation blocks unplanned lateral movement
- Whether only the allowed paths exist

IW04 (hardening/tuning) will refine:
- Deny rules
- Logging coverage
- Minimal egress strategy
