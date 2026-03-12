# Hardening — IW03 Defensive Baseline

This directory documents all hardening measures applied in Operation Iron Watch 03. Every measure here is a direct response to the gaps identified in IW02.

---

## Scope

| Layer | Scope |
|-------|-------|
| **Network** | DMZ segmentation, firewall rules on `sentry-gate01` |
| **Host** | SSH key-only authentication on all hosts |
| **Access Control** | Bastion model — management restricted to Safeguard Host only |

---

## 1. Network Hardening — DMZ Segmentation

### What Changed
`web-arm01` was moved from the flat LAN (192.168.0.0/24) into an isolated DMZ subnet (10.10.10.0/24). It can no longer freely communicate with LAN hosts.

### Enforcement Point
`sentry-gate01` sits between both zones and enforces a default-deny posture via UFW:
```bash
ufw default deny incoming
ufw default deny outgoing
ufw default deny routed
```

### Permitted Flows

| Source | Destination | Port | Purpose |
|--------|-------------|------|---------|
| Safeguard Host (LAN) | sentry-gate01 | 22/TCP | SSH management |
| web-arm01 (10.10.10.10) | sentry-gate01 (10.10.10.2) | 514/TCP | rsyslog log forwarding |
| sentry-gate01 (192.168.0.4) | soc-core03 (192.168.0.6) | 514/TCP | Graylog log ingestion |

> All other traffic between zones is denied. DMZ cannot initiate connections into the LAN beyond the log forwarding path.

---

## 2. Host Hardening — SSH Key-Only Authentication

### Applied To

| Host | IP |
|------|----|
| soc-core03 | 192.168.0.6 |
| sentry-gate01 | 192.168.0.4 (LAN) |
| web-arm01 | 10.10.10.10 |

### Configuration Applied
```bash
PasswordAuthentication no
PubkeyAuthentication yes
PermitRootLogin no
UsePAM no
```

### Critical Finding — Cloud-Init Override

On `web-arm01`, the file `/etc/ssh/sshd_config.d/50-cloud-init.conf` was **silently overriding all hardening applied to `/etc/ssh/sshd_config`**. SSH drop-in files in `/etc/ssh/sshd_config.d/` take precedence over the main config. The file was re-enabling password authentication despite the main config disabling it.

**Resolution:** File removed. Hardening confirmed effective after restart.

> This is a non-obvious bypass vector. Any SSH hardening effort must audit `/etc/ssh/sshd_config.d/` for drop-in overrides.

### Why This Matters
The IW02 SSH compromise succeeded because `web-arm01` accepted password-based authentication with weak credentials. SSH key-only auth makes brute-force attacks ineffective — no valid key means no access regardless of password attempts.

---

## 3. Access Control — Bastion Model

Only **Safeguard Host** (LAN) is permitted to SSH into any other host in the lab. No other machine — including those in the DMZ — can initiate SSH sessions toward LAN hosts.

### Enforcement
- UFW rule on `sentry-gate01` allows TCP/22 only from Safeguard Host
- SSH keys deployed from Safeguard Host only
- `web-arm01` in the DMZ has no SSH access path into the LAN

---

## 4. Hardening Checklist

| Item | Status | Host |
|------|--------|------|
| DMZ subnet created (10.10.10.0/24) | ✅ Done | sentry-gate01 / web-arm01 |
| TP-Link TL-SF1005D switch installed for DMZ isolation | ✅ Done | Physical |
| sentry-gate01 deployed with dual-NIC | ✅ Done | sentry-gate01 |
| UFW default deny (incoming/outgoing/routed) | ✅ Done | sentry-gate01 |
| IP forwarding enabled | ✅ Done | sentry-gate01 |
| Netplan static IP — persistent across reboots | ✅ Done | sentry-gate01 |
| SSH restricted to Safeguard Host only | ✅ Done | sentry-gate01 |
| SSH key-only auth | ✅ Done | All hosts |
| PermitRootLogin disabled | ✅ Done | All hosts |
| UsePAM no | ✅ Done | All hosts |
| 50-cloud-init.conf override removed | ✅ Done | web-arm01 |
| UFW allow-list for log forwarding | ✅ Done | sentry-gate01 |
| rsyslog installed on web-arm01 | ✅ Done | web-arm01 |
| rsyslog relay configured (web-arm01 → sentry-gate01 → soc-core03) | ✅ Done | web-arm01 / sentry-gate01 |
| Suricata installed on web-arm01 | ✅ Done | web-arm01 |
| Suricata EVE JSON output verified | ✅ Done | web-arm01 |
| Graylog inputs configured (auth.log, access.log, EVE JSON) | ✅ Done | soc-core03 |

---

## What Comes Next

Hardening is complete and validated. The log pipeline configuration is documented in `04-log-pipeline/`.

IW04 will validate this hardened baseline by running controlled reconnaissance and initial access attempts against the IW03 architecture — testing whether segmentation holds and whether Graylog surfaces the activity.
