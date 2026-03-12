# SYN Flood Detection — L4 Transport Layer

**Rule ID:** IW03-DDoS-L4 / Suricata sid:9000002  
**Layer:** Transport (L4)  
**Log Source:** Suricata EVE JSON (`event_type:flow`) + Suricata alert (`event_type:alert`)  
**Status:** ✅ Validated — 2026-03-12

---

## Attack Description

A SYN flood exploits the TCP three-way handshake. The attacker sends a large volume of SYN packets but never completes the handshake with the final ACK. This exhausts the server's connection table with half-open connections, denying service to legitimate users.

---

## Detection Logic — Dual Layer

SYN flood detection in IW03 uses two complementary mechanisms. This dual-layer approach emerged from a finding during implementation (see below).

### Layer 1 — Graylog Flow Counting (Primary)

**Why it works for SYN:** When SYN packets target a **closed port**, the server responds with RST immediately. Each RST closes the flow instantly, causing Suricata to write a separate `flow` record per packet. This produces the high flow count that Graylog's threshold logic needs.

| Parameter | Value |
|-----------|-------|
| **Filter** | `event_type:flow AND proto:TCP` |
| **Aggregation** | `count()` |
| **Threshold** | `> 100` |
| **Time window** | 1 minute |
| **Group-by** | `src_ip` |
| **Priority** | High |

**Limitation:** If SYN packets target an **open port**, half-open connections time out slowly (default ~60s). Flow records accumulate slowly — detection is delayed or missed entirely. This is why the Suricata rule exists as a backup.

### Layer 2 — Suricata Threshold Rule (Reliable Fallback)

Operates at the packet level — evaluates the SYN rate directly in the IDS, independent of flow closure behavior.

```suricata
alert tcp any any -> $HOME_NET 80 (msg:"IW03 - SYN Flood Detected"; flags:S; threshold:type threshold, track by_src, count 100, seconds 60; sid:9000002; rev:1;)
```

**Keyword breakdown:**
- `flags:S` — match TCP SYN flag only (the flood signature)
- `threshold:type threshold` — fires once per 100 matching packets per time window per source
- `track by_src` — evaluated independently per attacker IP
- `sid:9000002` — custom local rule ID

**Note:** Suricata warns `SYN-only to port(s) 80:80 w/o direction specified, disabling for toclient direction` — this is expected and harmless. SYN flood packets travel client → server only.

---

## Relevant Graylog Fields

| Field | Example Value | Source |
|-------|--------------|--------|
| `event_type` | `flow` / `alert` | Suricata EVE JSON extractor |
| `proto` | `TCP` | EVE JSON |
| `src_ip` | `10.10.10.1` | EVE JSON |
| `dest_ip` | `10.10.10.10` | EVE JSON |
| `alert_signature` | `IW03 - SYN Flood Detected` | EVE JSON alert block |
| `alert_signature_id` | `9000002` | EVE JSON alert block |
| `alert_severity` | `3` | EVE JSON alert block |

---

## Threshold Rationale

100 TCP SYN packets per minute from a single source is abnormal in any home lab scenario. Legitimate TCP connection rates are orders of magnitude lower. In production this threshold would be calibrated against baseline traffic — but for IW03 it reliably fires against flood traffic without false positives.

---

## Implementation Finding

**Flow counting works for SYN on closed ports.** Testing was performed with `hping3 -S -p 9999` — port 9999 is not listening on `web-arm01`, so every SYN received an immediate RST response. Each RST closed the flow, and Suricata wrote ~139 separate flow records in under a minute — well above the threshold.

This is an important nuance: the same detection logic that fails for ICMP works for SYN in the right conditions. The Suricata rule provides detection regardless of port state.

---

## Known Gaps

- **Distributed SYN flood:** Multiple source IPs each below threshold — group-by src_ip means coordinated floods from many sources are harder to surface.
- **Open port SYN flood:** Graylog flow counting is unreliable. The Suricata rule covers this case.

---

## Validation

**Test method:** `sudo hping3 -S -p 9999 -c 150 --fast 10.10.10.10` from Safeguard Host

| Field | Value |
|-------|-------|
| src_ip | 10.10.10.1 |
| dest_ip | 10.10.10.10 |
| count() | 139 |
| Timestamp | 2026-03-12 05:29:07 |
| Detection method | Graylog flow counting (closed port) |

**Evidence:** `evidences/SYN_and_ICMP_Flood_Event_Detection.png`
