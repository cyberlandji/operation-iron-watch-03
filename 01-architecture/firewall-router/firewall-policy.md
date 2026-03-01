# Firewall Policy Baseline (UFW) — Default Deny

## Purpose
Establish a **default deny** baseline on `sentry-gate01` so that:
- only explicitly allowed traffic is possible
- management access is tightly controlled
- future detection/monitoring operates on clear, intentional flows

## Baseline posture (LOCKED)
UFW is enabled with:
- **Default deny incoming**
- **Default deny outgoing**
- **Default deny routed (forwarded)**

This means:
- `sentry-gate01` is not a transparent router
- nothing crosses zones until allow rules exist

## Management plane rule (LOCKED)
SSH to `sentry-gate01` is restricted to Safeguard Host only.

Example intent:
- Allow TCP/22 from Safeguard Host IP (e.g., `192.168.0.7`)
- Deny SSH from all other sources (including DMZ and INTERNAL)

Rationale:
- Safeguard Host is the bastion / admin entrypoint
- reduces attack surface on management plane
- prevents opportunistic admin access from other LAN devices

## Forwarding rules (not yet added)
At this stage:
- routed traffic is denied by default
- DMZ <-> INTERNAL forwarding is not allowed

Planned next step:
- introduce minimal allow-lists for specific service needs
- log/monitor allowed and denied flows

## Logging intent (future enhancement)
To support detection engineering later in IW03:
- enable visibility into allowed/denied connections (UFW logs and/or packet inspection)
- correlate firewall decisions with SIEM telemetry

EOF
