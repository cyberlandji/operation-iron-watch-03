# HTTP Flood Detection — L7 Application Layer

**Rule ID:** IW03-DDos-L7  
**Layer:** Application (L7)  
**Log Source:** Apache `access.log` → rsyslog → sentry-gate01 → Graylog  
**Status:** ✅ Validated — 2026-03-12

---

## Attack Description

An HTTP flood is a Layer 7 volumetric attack where an attacker sends a high volume of HTTP requests to exhaust web server resources. Unlike SYN or ICMP floods, HTTP floods complete the full TCP handshake and send legitimate-looking requests — making them invisible to network-layer detection.

This is the one DDoS vector where Apache `access.log` is the correct detection source: every HTTP request generates a log entry regardless of server response.

---

## Detection Logic

### Why Flow Counting Works Here

Each HTTP request opens a separate TCP connection to the web server, generates an `access.log` entry, and closes. This means 1,000 HTTP requests → ~1,000 separate log entries in Graylog. Flow-count thresholds are valid and effective at L7.

### Graylog Event Definition

| Parameter | Value |
|-----------|-------|
| **Filter** | `event_type:http` |
| **Aggregation** | `count()` |
| **Threshold** | `> 50` |
| **Time window** | 1 minute |
| **Group-by** | `src_ip` |
| **Priority** | High |

**Filter & Aggregation** type event definition — Graylog counts matching messages per source IP within the rolling time window and fires when the threshold is crossed.

---

## Relevant Graylog Fields

| Field | Example Value | Source |
|-------|--------------|--------|
| `event_type` | `http` | Grok extractor on access.log |
| `src_ip` | `10.10.10.1` | Grok extractor |
| `dest_ip` | `10.10.10.10` | Grok extractor |
| `http_status` | `200`, `404` | Grok extractor |
| `request` | `GET / HTTP/1.1` | Grok extractor |
| `source` | `web-arm01` | rsyslog tag |

---

## Threshold Rationale

50 requests per minute from a single source IP is well above normal browsing behavior in a home lab environment. In production this threshold would be significantly higher — but for IW03 it provides a reliable detection signal without false positives against legitimate traffic.

---

## Known Gaps

- **Slow-and-low attacks (Slowloris):** Sends very few, very slow HTTP requests to hold connections open. Falls below the volume threshold entirely. Not detected by this rule.
- **Distributed HTTP flood:** Multiple source IPs each sending below threshold. Group-by src_ip means distributed floods are harder to detect without correlation across IPs.

---

## Validation

**Test method:** Apache benchmark / curl loop from Safeguard Host (10.10.10.1) targeting web-arm01 (10.10.10.10)

| Field | Value |
|-------|-------|
| src_ip | 10.10.10.1 |
| dest_ip | 10.10.10.10 |
| count() | 60 |
| Timestamp | 2026-03-12 03:52:46 |
| Events fired | 2 (within 1-hour window) |

**Evidence:** `evidences/HTTP_Flood_Event_Detection.png`, `evidences/HTTP_Flood_Event_Details.png`
