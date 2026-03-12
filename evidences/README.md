# Evidences — Operation Iron Watch 03

This directory contains all evidence collected during Operation Iron Watch 03, organized by category. Screenshots are named and ordered to reflect the actual investigation and validation flow.

---

## SSH Hardening

Located in `ssh-hardening/` — see `ssh-hardening/README.md` for full context and investigation narrative.

| File | Description |
|------|-------------|
| `1_ssh_hardening1.png` | sshd_config — PasswordAuthentication no, PermitRootLogin no |
| `1_ssh_hardening2.png` | sshd_config — UsePAM no |
| `2_ssh_bypass.png` | Password auth bypass discovered — login succeeded despite config changes |
| `3_ssh_bypass_rootcause.png` | 50-cloud-init.conf identified as override source |
| `4_ssh_bypass_rootcause_fix.png` | 50-cloud-init.conf corrected |
| `5_ssh_hardening_final.png` | Permission denied (publickey) confirmed — hardening complete |

---

## Log Pipeline & Graylog

| File | Description |
|------|-------------|
| `BASELOG_VIEWS.png` | Graylog saved searches — BASELOG VIEW and BASELOG_FILTERED VIEW |
| `HttpApache_logs_from_webarm01.png` | Apache access.log entries structured in Graylog |
| `BASELOG_logs_from_Safeguard.png` | Graylog stream showing logs from Safeguard Host |
| `Event_flow_proto_UDP.png` | Suricata EVE flow events — UDP protocol visible in Graylog |

---

## DDoS Detection Suite — Validation

### HTTP Flood (L7)
| File | Description |
|------|-------------|
| `HTTP_Flood_Event_Detection_proof.png` | Graylog Events tab — two HTTP Flood events fired, 2026-03-12 03:52 |
| `HTTP_Flood_Event_Details.png` | Event detail — src_ip 10.10.10.1, count()=60, priority High |

### ICMP Flood (L3)
| File | Description |
|------|-------------|
| `Suricata_alert_log.png` | eve.json tail — alert records firing during ICMP flood test |
| `SURICATA_Alert.png` | Graylog stream — event_type:alert ICMP events from 10.10.10.1 |
| `ICMP_Alert_signature.png` | Graylog message detail — alert_signature, sid 9000001, structured fields |
| `ICMP_Flood_Event_Detection.png` | Graylog Events — IW03 - ICMP Flood Detected, count()=28, 2026-03-12 05:23 |

### SYN Flood (L4) + Full Suite Confirmation
| File | Description |
|------|-------------|
| `SYN_and_ICMP_Flood_Event_Detection.png` | Graylog Events — all three detections confirmed in single view |

---

## Detection Rule Troubleshooting

| File | Description |
|------|-------------|
| `adding_fields_alert.png` | Graylog — alert fields configuration during Suricata EVE JSON extractor setup |
| `One_ICMP_captured.png` | Single ICMP flow record — demonstrates why flow counting fails for ICMP flood detection |
| `ICMP.png` | ICMP traffic visible in Graylog stream — initial test before Suricata rule redesign |

---

## Note on Naming

Files are prefixed or grouped where possible to reflect chronological order within each investigation phase. See the individual rule files in `05-detection-rules/` for cross-references to specific evidence files per detection use case.
