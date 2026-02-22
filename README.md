# 🔐 Operation Iron Watch 03  
## Post-Compromise Detection, Investigation & Hardening

Operation Iron Watch 03 continues directly from the compromise observed in Iron Watch ß2.

Despite early reconnaissance detections, the attacker successfully gained valid SSH access to the web server due to weak credentials and incomplete baselines. This operation focuses on what happens *after* initial access: detecting attacker activity, investigating their behavior, and hardening the environment to prevent recurrence.

---

## 🎯 Objectives

- Detect post-compromise host activity
- Identify privilege escalation attempts
- Observe lateral movement indicators
- Detect defense evasion techniques
- Perform investigation and timeline reconstruction
- Apply containment, eradication, and hardening measures

---

## 🧠 Assumed Context

The following events are assumed to have already occurred:
- Reconnaissance activity detected (scans, abnormal HTTP requests)
- SSH brute-force succeeded using weak credentials
- Attacker gained shell access on `web-arm01`
- Logging existed but baselines were insufficient

Iron Watch 03 does **not** replay the attack — it responds to its consequences.

---

## 🛠️ Tooling

- Linux (Ubuntu Server)
- SSH
- Apache
- Centralized logging (Graylog / SIEM)
- System logs (auth.log, syslog, bash history)
- Network telemetry

---

## 📂 Structure

See each directory for detailed documentation:
- Detection use cases
- Investigation workflows
- Response & hardening actions
- Supporting evidence

---

## 🏁 Outcome

Iron Watch 03 demonstrates real SOC capability:
- Accepting compromise as reality
- Investigating attacker behavior
- Improving detection maturity
- Strengthening defensive posture
