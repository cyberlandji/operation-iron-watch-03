# Log Pipeline — Configuration & Implementation

This directory documents the log pipeline implemented in Operation Iron Watch 03. Structured, actionable log data flows from `web-arm01` through `sentry-gate01` into Graylog on `soc-core03`.

---

## Overview

In IW02, only `access.log` was ingested into Graylog — manually. IW03 introduces a proper automated log pipeline covering three log sources across two network zones, relayed through the firewall layer.

### Log Sources

| Log | Host | Format | Content |
|-----|------|--------|---------|
| `auth.log` | web-arm01 | Syslog | SSH attempts, authentication events |
| `access.log` | web-arm01 | Apache Combined Log | HTTP requests, status codes, IPs |
| `eve.json` | web-arm01 | JSON (Suricata EVE) | Network alerts, flows, protocol metadata |

---

## Architecture
```
web-arm01 (10.10.10.10)
    │
    │  rsyslog forwards via imfile:
    │  - auth.log       → syslog, 514/TCP
    │  - access.log     → syslog, 514/TCP
    │  - eve.json       → syslog, 514/TCP
    │
    ▼
sentry-gate01 (10.10.10.2) — rsyslog relay
    │
    │  receives all streams on :514
    │  forwards to Graylog
    │
    ▼
soc-core03 (192.168.0.6)
    └── Graylog
            ├── Syslog TCP input → auth.log + access.log
            └── Syslog TCP + JSON extractor → eve.json
```

---

## rsyslog Configuration

### web-arm01 — Forward to sentry-gate01
```bash
# /etc/rsyslog.d/99-forward-to-sentry.conf

module(load="imfile")

# auth.log
input(type="imfile"
      File="/var/log/auth.log"
      Tag="auth"
      Severity="info"
      Facility="authpriv")

# Apache access.log
input(type="imfile"
      File="/var/log/apache2/access.log"
      Tag="apache-access"
      Severity="info"
      Facility="local6")

local6.* @@10.10.10.2:514

# Suricata EVE JSON
input(type="imfile"
      File="/var/log/suricata/eve.json"
      Tag="suricata-eve"
      Severity="info"
      Facility="local7")

local7.* @@10.10.10.2:514
```

> `@@` = TCP. TCP preferred over UDP for reliable log delivery.

### sentry-gate01 — Relay to Graylog
```bash
# /etc/rsyslog.d/99-relay-to-graylog.conf

module(load="imtcp")
input(type="imtcp" port="514")

*.* @@192.168.0.6:514
```

---

## Graylog Inputs

| Input Type | Port | Receives |
|------------|------|---------|
| Syslog TCP | 514 | auth.log, access.log, Suricata EVE JSON |

### Extractors Configured

| Field | Extractor Type | Applied To |
|-------|---------------|------------|
| `event_type`, `src_ip`, `dest_ip`, `proto` | JSON extractor | Suricata EVE JSON messages |
| `src_ip`, `dest_ip`, `http_status`, `request` | Grok extractor | Apache access.log messages |
| `alert_signature`, `alert_severity`, `alert_signature_id` | JSON extractor | Suricata alert events |

---

## Firewall Allow-List (sentry-gate01 UFW)
```bash
# Allow rsyslog from web-arm01 into sentry-gate01
ufw allow from 10.10.10.10 to any port 514 proto tcp

# Allow sentry-gate01 to forward logs to Graylog
ufw allow out to 192.168.0.6 port 514 proto tcp
```

---

## Troubleshooting Notes

**rsyslog imfile module syntax error** — A legacy vs modern module syntax conflict in the rsyslog config on `web-arm01` prevented log forwarding initially. Corrected by using the modern `module(load="imfile")` syntax consistently throughout the config file. Mixing legacy and RainerScript syntax in the same file causes silent failures.

**Graylog `http_publish_uri` mismatch** — After VM migration from IW02, Graylog still had the old host-only IP set as `http_publish_uri`. Updated to `192.168.0.6` — this was blocking the web interface from loading correctly.

---

## Verification

Pipeline verified end-to-end by:

1. SSH login attempts on `web-arm01` → `auth.log` entries confirmed in Graylog with `facility: security/authorization`
2. HTTP requests to Apache → `access.log` entries confirmed with structured fields: `src_ip`, `http_status`, `request`
3. ICMP/SYN/HTTP flood test traffic → Suricata EVE alerts confirmed in Graylog with `event_type: alert`, `alert_signature`, `src_ip`, `dest_ip`
4. All three DDoS event definitions fired successfully against live traffic

---

## Implementation Status

| Step | Status |
|------|--------|
| rsyslog installed on web-arm01 | ✅ Done |
| Suricata installed on web-arm01 | ✅ Done |
| rsyslog config on web-arm01 (imfile → sentry-gate01) | ✅ Done |
| rsyslog config on sentry-gate01 (relay → Graylog) | ✅ Done |
| UFW allow-list on sentry-gate01 for log forwarding | ✅ Done |
| Graylog Syslog TCP input configured | ✅ Done |
| Graylog JSON + Grok extractors configured | ✅ Done |
| Pipeline verified end-to-end | ✅ Done |
