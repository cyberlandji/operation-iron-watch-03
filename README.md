# 🔐 Operation Iron Watch 03
## DMZ Hardening, Log Pipeline & Detection Engineering

[![Status](https://img.shields.io/badge/status-complete-green)](https://github.com/cyberlandji/operation-iron-watch-03)
[![Lab Series](https://img.shields.io/badge/series-Operation%20Iron%20Watch-blue)](https://github.com/cyberlandji)
[![Focus](https://img.shields.io/badge/focus-Hardening%20%26%20Detection-teal)](https://github.com/cyberlandji/operation-iron-watch-03)

---

## 📌 Overview

Operation Iron Watch 03 is the hardening and detection engineering phase of the Iron Watch series.

Iron Watch 02 exposed a critical gap: the SSH compromise of `web-arm01` was **completely invisible** on the SIEM because `auth.log` was never ingested. The attacker moved freely while Graylog only watched HTTP 404 spikes. IW03 is the direct response to that failure.

This operation introduces a proper **DMZ architecture**, a **firewall relay layer**, an **expanded log pipeline**, and a structured **DDoS detection suite** built across three network layers. No attacker is simulated — this is pure hardening and proactive detection engineering.

---

## 🎯 Objectives

- Design and implement a segmented DMZ (web-arm01 isolated from LAN)
- Deploy `sentry-gate01` as a firewall VM and rsyslog relay between DMZ and LAN
- Enforce SSH key-only authentication across all hosts
- Ingest `auth.log`, `access.log`, and Suricata EVE JSON into Graylog
- Build a three-layer DDoS Detection Suite in Graylog:
  - **L3** — ICMP Flood (firewall / Suricata)
  - **L4** — SYN Flood (Suricata EVE JSON)
  - **L7** — HTTP Flood (Apache access.log)
- Enrich alerts with severity levels via Graylog pipeline rules
- Document every architectural decision and detection logic

---

## 🏗️ Architecture

### Network Zones

| Zone | Subnet | Purpose |
|------|--------|---------|
| LAN Zone | 192.168.0.0/24 | Trusted internal network |
| DMZ | 10.10.10.0/24 | Untrusted public-facing zone |

### Hosts

| Host | IP | Role |
|------|----|------|
| Router | 192.168.0.1 | Home router (CGNAT) |
| Safeguard Host | 192.168.0.7 | Physical host running VMs |
| soc-core04 (VM) | 192.168.0.5 | SIEM — Graylog |
| sentry-gate01 (VM) | wlo1: 192.168.0.x / enp3s0: 10.10.10.x | Firewall VM + rsyslog relay |
| web-arm01 (Pi) | 10.10.10.10 | Web server + Suricata IDS — in DMZ |

### Log Flow

```
web-arm01 (rsyslog)
    │
    │  auth.log / access.log / Suricata EVE JSON
    ▼
sentry-gate01 (rsyslog relay)
    │
    │  forwarded logs → Graylog input
    ▼
soc-core04 (Graylog)
```

> See `01-architecture/` for the full draw.io diagram and zone documentation.

---

## 🛠️ Tooling

| Tool | Host | Role |
|------|------|------|
| Graylog | soc-core04 | SIEM — log ingestion, alerting, pipeline enrichment |
| Suricata | web-arm01 | Network IDS — EVE JSON output |
| rsyslog | web-arm01 + sentry-gate01 | Log forwarding and relay |
| Apache2 | web-arm01 | Web server — access.log source |
| iptables/nftables | sentry-gate01 | Firewall rules — DMZ enforcement |
| draw.io | — | Architecture diagrams |
| SSH (key-only) | all hosts | Hardened remote access |

---

## 📂 Structure

```
operation-iron-watch-03/
│
├── 01-architecture/       # Network diagram, zone design, IP plan
├── 02-context/            # IW02 lessons learned, threat model, design decisions
├── 03-hardening/          # SSH hardening, DMZ setup, firewall rules
├── 04-log-pipeline/       # rsyslog configs, Graylog inputs, EVE JSON parsing
├── 05-detection-rules/    # DDoS detection suite — L3/L4/L7 rules + pipeline enrichment
├── evidences/             # Screenshots, alert captures, log samples
├── LICENSE
└── README.md
```

---

## 🔍 Detection Suite — DDoS Detection

IW03 implements a three-layer DDoS Detection Suite in Graylog, covering distinct attack vectors:

| Rule | Layer | Source | Detection Logic |
|------|-------|--------|----------------|
| SYN Flood | L4 | Suricata EVE JSON | High rate of SYN packets without ACK completion |
| ICMP Flood | L3 | Suricata / firewall | Abnormal volume of ICMP echo requests |
| HTTP Flood | L7 | Apache access.log | Spike in HTTP requests per source IP per time window |

All alerts are enriched with severity levels (`LOW / MEDIUM / HIGH / CRITICAL`) via Graylog pipeline rules.

> See `05-detection-rules/` for full rule logic and pipeline configurations.

---

## 🔗 Iron Watch Series

| Episode | Focus | Status |
|---------|-------|--------|
| [Iron Watch 01](https://github.com/cyberlandji/operation-iron-watch-01) | Foundational SOC — Snort IDS, network visibility | ✅ Complete |
| [Iron Watch 02](https://github.com/cyberlandji/operation-iron-watch-02) | Graylog SIEM — web enumeration detection | ✅ Complete |
| **Iron Watch 03** | **DMZ hardening, log pipeline, detection engineering** | 🔄 In Progress |
| Iron Watch 04 | Attack validation — Kali recon & initial access against IW03 | 🔜 Planned |

---

## 👤 Author

**cyberlandji** — Blue Team Practitioner | ISC2 CC | CompTIA Security+ (in progress)

Portfolio: [cyberlandji.com](https://cyberlandji.com) · GitHub: [github.com/cyberlandji](https://github.com/cyberlandji)

---

*Part of the Operation Iron Watch home lab series — building real SOC capability from scratch.*
