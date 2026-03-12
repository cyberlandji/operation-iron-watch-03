# Routing (IP Forwarding) — Philosophy & Controls

This file documents the routing philosophy and IP forwarding decisions for `sentry-gate01` in Operation Iron Watch 03.

---

## Why Routing Matters Here

Without routing, isolated zones cannot communicate even when needed. With routing, traffic can traverse zones — but must be controlled.

---

## Explicit Routing Principle

Routing is treated as a **capability**, not a default behavior:
- The kernel forwards packets **only when intentionally enabled**
- Routing is paired with firewall policy to enforce **least privilege**

---

## Implementation

IP forwarding enabled and persisted in `/etc/sysctl.conf`:
```bash
net.ipv4.ip_forward = 1
```

`sentry-gate01` is not a transparent router — nothing crosses zones without an explicit UFW allow rule.

---

## Security Stance

Even with IP forwarding enabled:
- Default-deny UFW posture remains the primary enforcement layer
- No DMZ → LAN forwarding is permitted outside the defined allow-list below

---

## Permitted Flows

| Source | Destination | Port | Protocol | Purpose |
|--------|-------------|------|----------|---------|
| web-arm01 (10.10.10.10) | sentry-gate01 (10.10.10.2) | 514 | TCP | rsyslog — DMZ log ingestion |
| sentry-gate01 (192.168.0.4) | soc-core03 (192.168.0.6) | 514 | TCP | rsyslog — relay to Graylog |
| Safeguard Host (LAN) | sentry-gate01 | 22 | TCP | SSH management (LAN only) |

> All other cross-zone traffic is denied by default.

---

## What Comes Next (IW04)

Routing alone is not enough. IW04 (attack validation) will test:
- Whether segmentation blocks unplanned lateral movement
- Whether only the defined allowed paths exist in practice
- Whether Suricata and Graylog surface the attempts in real time

---

## Status

- ✅ IP forwarding enabled and persisted via sysctl.conf
- ✅ UFW default-deny posture enforced
- ✅ Permitted flows implemented and validated end-to-end
