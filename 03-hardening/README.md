# Hardening — IW03 Defensive Baseline

This directory documents all hardening measures applied in Operation Iron Watch 03. Every measure here is a direct response to the gaps identified in IW02.

---

## Scope

Hardening in IW03 covers three layers:

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
# Applied on sentry-gate01
ufw default deny incoming
ufw default deny outgoing
ufw default deny routed
```

### Permitted Flows (Explicit Allow-List)

| Source | Destination | Port | Purpose |
|--------|-------------|------|---------|
| Safeguard Host (192.168.0.7) | sentry-gate01 | 22/TCP | SSH management |
| web-arm01 (10.10.10.10) | sentry-gate01 | 514/TCP | rsyslog log forwarding |
| sentry-gate01 | soc-core04 (192.168.0.5) | 514/TCP or 12201/UDP | Graylog log ingestion |

> All other traffic between zones is denied. DMZ cannot initiate connections into the LAN beyond the log forwarding path.

---

## 2. Host Hardening — SSH Key-Only Authentication

### Applied To
All hosts in the lab:

| Host | IP |
|------|----|
| Safeguard Host | 192.168.0.7 |
| soc-core04 | 192.168.0.5 |
| sentry-gate01 | 192.168.0.x (LAN) |
| web-arm01 | 10.10.10.10 |

### Configuration Applied

SSH password authentication disabled on all hosts via `/etc/ssh/sshd_config`:

```bash
PasswordAuthentication no
PubkeyAuthentication yes
PermitRootLogin no
```

Reloaded with:

```bash
sudo systemctl reload sshd
```

### Why This Matters

The IW02 SSH compromise succeeded because `web-arm01` accepted password-based authentication with weak credentials. SSH key-only auth makes brute-force attacks ineffective — no valid key means no access, regardless of how many password attempts are made.

---

## 3. Access Control — Bastion Model

### Principle
Only **Safeguard Host (192.168.0.7)** is permitted to SSH into any other host in the lab. No other machine — including those in the DMZ — can initiate SSH sessions toward LAN hosts.

### Enforcement
- UFW rule on `sentry-gate01` allows TCP/22 only from 192.168.0.7
- SSH keys are only deployed from Safeguard Host
- `web-arm01` in the DMZ has no SSH access path into the LAN

---

## 4. Hardening Checklist

| Item | Status | Host |
|------|--------|------|
| DMZ subnet created (10.10.10.0/24) | ✅ Done | sentry-gate01 / web-arm01 |
| sentry-gate01 deployed with dual-NIC | ✅ Done | sentry-gate01 |
| UFW default deny (incoming/outgoing/routed) | ✅ Done | sentry-gate01 |
| IP forwarding enabled | ✅ Done | sentry-gate01 |
| SSH restricted to Safeguard Host only | ✅ Done | sentry-gate01 |
| rsyslog installed on web-arm01 | ✅ Done | web-arm01 |
| Suricata installed on web-arm01 | ✅ Done | web-arm01 |
| SSH key-only auth | ✅ Done | All hosts |
| PermitRootLogin disabled | ✅ Done| All hosts |
| UsePAM no | ✅ Done | All hosts |
| UFW allow-list for log forwarding | ✅ Done | sentry-gate01 |
| rsyslog relay configured (web-arm01 → sentry-gate01 → soc-core04) | 🔜 Planned | web-arm01 / sentry-gate01 |
| Suricata EVE JSON output verified | 🔜 Planned | web-arm01 |
| Graylog inputs configured (auth.log, access.log, EVE JSON) | 🔜 Planned | soc-core04 |

---

## What Comes Next

Once the firewall allow-list for log forwarding is confirmed and tested, hardening is complete. The log pipeline configuration is documented in `04-log-pipeline/`.

IW04 will then validate this hardened baseline by running controlled reconnaissance and initial access attempts against the IW03 architecture.
