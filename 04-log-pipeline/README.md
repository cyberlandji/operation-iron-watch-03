# Log Pipeline — Plan & Configuration

This directory documents the planned log pipeline for Operation Iron Watch 03. The goal is to feed structured, actionable log data from `web-arm01` through `sentry-gate01` into Graylog on `soc-core04`.

> ⚠️ This document reflects the **planned architecture**. Configuration details will be updated as implementation progresses and real-world adjustments are made.

---

## Overview

In IW02, only `access.log` was ingested into Graylog — and even that was done manually. IW03 introduces a proper, automated log pipeline covering three log sources across two network zones, relayed through the firewall layer.

### Log Sources

| Log | Host | Format | Content |
|-----|------|--------|---------|
| `auth.log` | web-arm01 | Syslog | SSH attempts, authentication events |
| `access.log` | web-arm01 | Apache Combined Log | HTTP requests, status codes, IPs |
| `eve.json` | web-arm01 | JSON (Suricata EVE) | Network alerts, protocol metadata |

---

## Planned Architecture

```
web-arm01 (10.10.10.10)
    │
    │  rsyslog forwards:
    │  - auth.log       → syslog format, port 514/TCP
    │  - access.log     → syslog format, port 514/TCP
    │  - eve.json       → JSON format, port 514/TCP (or dedicated port)
    │
    ▼
sentry-gate01 (rsyslog relay)
    │
    │  receives all log streams
    │  forwards to Graylog on soc-core04
    │
    ▼
soc-core04 (192.168.0.5)
    │
    └── Graylog
            ├── Syslog UDP/TCP input → auth.log + access.log
            └── GELF or Raw/Plaintext JSON input → eve.json
```

---

## Planned rsyslog Configuration

### On web-arm01 — Forward Logs to sentry-gate01

```bash
# /etc/rsyslog.d/99-forward-to-sentry.conf

# Forward auth log
if $programname == 'sshd' then @@<sentry-gate01-DMZ-IP>:514

# Forward Apache access log (via imfile module)
module(load="imfile")

input(type="imfile"
      File="/var/log/apache2/access.log"
      Tag="apache-access"
      Severity="info"
      Facility="local6")

local6.* @@<sentry-gate01-DMZ-IP>:514

# Forward Suricata EVE JSON (via imfile module)
input(type="imfile"
      File="/var/log/suricata/eve.json"
      Tag="suricata-eve"
      Severity="info"
      Facility="local7")

local7.* @@<sentry-gate01-DMZ-IP>:514
```

> `@@` means TCP. `@` means UDP. TCP preferred for reliability.

### On sentry-gate01 — Receive and Relay to Graylog

```bash
# /etc/rsyslog.d/99-relay-to-graylog.conf

# Enable TCP input on port 514
module(load="imtcp")
input(type="imtcp" port="514")

# Forward everything received to Graylog on soc-core04
*.* @@192.168.0.5:514
```

---

## Planned Graylog Inputs

| Input Type | Port | Receives |
|------------|------|---------|
| Syslog TCP | 514 | auth.log, access.log |
| Raw/Plaintext TCP or GELF | 12201 | Suricata EVE JSON |

> Exact input types may be adjusted during implementation depending on how Graylog parses the forwarded EVE JSON stream.

---

## Planned Firewall Allow-List (sentry-gate01 UFW)

These rules need to be added to `sentry-gate01` to permit the log forwarding flow:

```bash
# Allow rsyslog from web-arm01 into sentry-gate01
ufw allow from 10.10.10.10 to any port 514 proto tcp

# Allow sentry-gate01 to forward logs to Graylog on soc-core04
ufw allow out to 192.168.0.5 port 514 proto tcp
```

---

## Verification Plan

Once implemented, the pipeline will be verified by:

1. Generating test SSH login attempts on `web-arm01` → confirm `auth.log` entries appear in Graylog
2. Sending HTTP requests to Apache on `web-arm01` → confirm `access.log` entries appear in Graylog
3. Triggering a Suricata alert → confirm EVE JSON event appears in Graylog
4. Confirming all three log sources produce structured, searchable fields in Graylog

---

## Implementation Status

| Step | Status |
|------|--------|
| rsyslog installed on web-arm01 | ✅ Done |
| Suricata installed on web-arm01 | ✅ Done |
| rsyslog config on web-arm01 (forward to sentry-gate01) | 🔜 Planned |
| rsyslog config on sentry-gate01 (relay to Graylog) | 🔜 Planned |
| UFW allow-list on sentry-gate01 for log forwarding | 🔜 Planned |
| Graylog Syslog input configured | 🔜 Planned |
| Graylog EVE JSON input configured | 🔜 Planned |
| Pipeline verified end-to-end | 🔜 Planned |

> This document will be updated with actual configuration details once implementation is complete.
