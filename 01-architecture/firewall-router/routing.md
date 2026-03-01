# Routing (IP Forwarding) — Philosophy & Controls

## Why routing matters here
`sentry-gate01` sits between DMZ and INTERNAL.
Without routing, zones are isolated but cannot communicate even when needed.
With routing, traffic can traverse zones — but must be controlled.

## Explicit routing principle (LOCKED)
Routing is treated as a **capability**, not a default behavior:
- We enable the kernel to forward packets **only when we intentionally decide**
- We pair routing with firewall policy to ensure **least privilege**

## Implementation decision
- `net.ipv4.ip_forward` was enabled intentionally for IW03
- Persisted in `/etc/sysctl.conf`

Example line (persisted):
- `net.ipv4.ip_forward=1`

## Security stance
Even with IP forwarding enabled:
- default deny firewall posture remains the main enforcement layer
- forwarding between DMZ and INTERNAL is **not permitted until explicit allow rules are added**

## What comes next (IW03 steps)
Routing alone is not enough.
We will later implement controlled allow-lists such as:
- DMZ -> INTERNAL: only required destination ports (e.g., telemetry/log shipping)
- INTERNAL -> DMZ: only required management or monitoring flows (if needed)
- Any internet egress: explicit, minimal, logged

## Why this supports IW04/IW05
IW04 (attack validation) can show:
- whether segmentation blocks unintended lateral movement
- whether only the allowed paths exist

IW05 (tuning/hardening) will refine:
- deny rules
- logging coverage
- minimal egress strategy

EOF
