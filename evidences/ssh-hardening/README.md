# SSH Hardening — Evidence & Troubleshooting Log

**Host:** `web-arm01` (10.10.10.10)  
**Date:** 2026-03-06  
**Performed from:** Safeguard Host (192.168.0.7)

---

## Objective

Enforce SSH key-only authentication on `web-arm01` as part of the IW03 hardening baseline. Password-based SSH was the direct cause of the IW02 compromise — this closes that gap permanently.

---

## Changes Applied

### 1. SSH Key Pair Generated on Safeguard Host

A dedicated ed25519 key pair was generated specifically for the Safeguard Host → web-arm01 connection, following the principle of one key per host:

```bash
ssh-keygen -t ed25519 -C "safeguard-to-web-arm01" -f ~/.ssh/id_ed25519_web_arm01
```

Public key copied to web-arm01:

```bash
ssh-copy-id -i ~/.ssh/id_ed25519_web_arm01.pub webadmin@10.10.10.10
```

Key login verified before disabling password auth — critical step to avoid lockout.

---

### 2. Main sshd_config Changes (`/etc/ssh/sshd_config`)

Three directives uncommented and set:

| Directive | Value | Reason |
|-----------|-------|--------|
| `PasswordAuthentication` | `no` | Disable password-based SSH login |
| `PermitRootLogin` | `no` | Prevent direct root SSH access |
| `UsePAM` | `no` | Prevent PAM modules from bypassing PasswordAuthentication |

> `UsePAM no` is critical on Debian/Ubuntu — PAM can bypass `PasswordAuthentication no` through its own authentication modules if left enabled.

---

### 3. Troubleshooting — The sshd_config.d Override

**Problem:** After applying all three changes and reloading sshd, password authentication was still accepted.

**Investigation:** The main sshd_config contains:
```
Include /etc/ssh/sshd_config.d/*.conf
```

This means any `.conf` file in that directory can **override** settings in the main sshd_config.

**Root cause found:**
```
/etc/ssh/sshd_config.d/50-cloud-init.conf
```

This file contained:
```
PasswordAuthentication yes
```

This is a standard cloud-init file present on cloud-provisioned and some ARM/Pi images. It was silently overriding the main config.

**Fix applied:** Changed `PasswordAuthentication yes` → `PasswordAuthentication no` in `/etc/ssh/sshd_config.d/50-cloud-init.conf`.

> ⚠️ **Important note for future hardening:** Always check `/etc/ssh/sshd_config.d/` for override files after modifying the main sshd_config. On cloud instances (AWS, GCP, Azure) and some ARM images, `50-cloud-init.conf` will silently override your settings. Hardening is only complete when verified — not just configured.

---

### 4. Final Verification

sshd reloaded and tested:

```bash
sudo systemctl reload sshd
sudo systemctl status sshd
```

Password authentication rejection confirmed from Safeguard Host:

```bash
ssh -o PreferredAuthentications=password -o PubkeyAuthentication=no webadmin@10.10.10.10
# Result: Permission denied (publickey)
```

---

## Evidence Files

Files are ordered chronologically to reflect the actual investigation flow:

| # | File | Description |
|---|------|-------------|
| 1 | `1_ssh_hardening1.png` | sshd_config — first part of the file showing PasswordAuthentication no and PermitRootLogin no |
| 2 | `1_ssh_hardening2.png` | sshd_config — second part of the file showing UsePAM no, same config file continued |
| 3 | `2_ssh_bypass.png` | Password auth tested — bypass discovered, login succeeded despite config changes |
| 4 | `3_ssh_bypass_rootcause.png` | 50-cloud-init.conf identified — PasswordAuthentication yes overriding main config |
| 5 | `4_ssh_bypass_rootcause_fix.png` | 50-cloud-init.conf modified — PasswordAuthentication yes changed to no |
| 6 | `5_ssh_hardening_final.png` | sshd reloaded — Permission denied (publickey) confirmed, hardening complete |

---

## Hardening Status

| Item | Status |
|------|--------|
| SSH key pair generated (ed25519, dedicated) | ✅ Done |
| Public key copied to web-arm01 | ✅ Done |
| Key-based login verified | ✅ Done |
| PasswordAuthentication no (sshd_config) | ✅ Done |
| PermitRootLogin no (sshd_config) | ✅ Done |
| UsePAM no (sshd_config) | ✅ Done |
| 50-cloud-init.conf override fixed | ✅ Done |
| Password auth rejection verified | ✅ Done |
