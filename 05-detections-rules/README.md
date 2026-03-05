# Detection Rules — DDoS Detection Suite

This directory contains the detection rules built and validated during Operation Iron Watch 03. Rules are developed live against real log data flowing through the IW03 log pipeline — not written in advance.

> ⚠️ This document reflects the **planned detection scope**. Individual rule files will be added as each rule is implemented, tested, and validated in Graylog.

---

## Detection Philosophy

Rules in IW03 are built following this cycle:

1. **Understand the log source** — know exactly what fields Graylog produces for each input
2. **Write a candidate rule** — based on real observed traffic patterns
3. **Test against live data** — confirm the rule fires correctly and does not generate excessive false positives
4. **Document the rule** — logic, threshold rationale, severity level, and evidence
5. **Refine if needed** — adjust thresholds based on real environment noise

> A rule that is not tested is not a detection. It is a guess.

---

## Detection Suite Scope — DDoS Detection (Three Layers)

IW03 implements a structured DDoS Detection Suite covering three distinct network layers. Each layer targets a different attack vector and uses a different log source.

| Rule | Layer | Log Source | Detection Target | Status |
|------|-------|------------|-----------------|--------|
| HTTP Flood | L7 — Application | Apache `access.log` | Abnormal HTTP request spike per source IP per time window | 🔜 Planned |
| SYN Flood | L4 — Transport | Suricata EVE JSON | High rate of SYN packets without ACK completion | 🔜 Planned |
| ICMP Flood | L3 — Network | Suricata EVE JSON / firewall | Abnormal volume of ICMP echo requests | 🔜 Planned |

---

## Severity Enrichment

All alerts will be enriched with severity levels via **Graylog pipeline rules**:

| Severity | Meaning |
|----------|---------|
| `LOW` | Anomaly detected — below threshold, informational |
| `MEDIUM` | Threshold crossed — warrants attention |
| `HIGH` | Sustained or escalating pattern — likely attack |
| `CRITICAL` | Confirmed attack pattern — immediate action required |

> Severity thresholds will be calibrated against real traffic observed in the lab environment during implementation.

---

## Rule Files (Added During Implementation)

Rule documentation files will be added here as each rule is built and validated:

```
05-detection-rules/
├── README.md               ← this file
├── http-flood.md           ← L7 rule (added during implementation)
├── syn-flood.md            ← L4 rule (added during implementation)
└── icmp-flood.md           ← L3 rule (added during implementation)
```

Each rule file will document:
- Rule logic and Graylog query
- Log source and relevant fields
- Threshold rationale
- Severity mapping
- Test evidence (screenshots / log samples in `evidences/`)

---

## Implementation Status

| Item | Status |
|------|--------|
| Log pipeline delivering data to Graylog | 🔜 Planned |
| Graylog field mapping verified per log source | 🔜 Planned |
| HTTP Flood rule written and tested | 🔜 Planned |
| SYN Flood rule written and tested | 🔜 Planned |
| ICMP Flood rule written and tested | 🔜 Planned |
| Severity enrichment pipeline configured | 🔜 Planned |
| All rules validated with evidence | 🔜 Planned |

> Rules will be developed live during implementation. Each rule goes through the full write → test → validate → document cycle before being marked ✅ Done.
