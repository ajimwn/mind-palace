---
type: tool
room: blue
tool_name: "log-hunting"
version: "grep / awk / sed"
platform: [Linux, Windows]
phase: [Detection, Log Analysis, Triage]
tags: [blue/tool, blue/log-analysis, grep, awk, linux-logs, auth-log]
date_added: 2026-10-30
last_reviewed: 2026-10-30
---

# Log Hunting (grep & awk)

> [!quote]
> *"Never trust to general impressions, my boy, but concentrate yourself upon details."*
> — *A Case of Identity*

**Room:** Blue — Instruments | **Role:** High-Speed CLI Log Parsing and Anomaly Hunting | **Platform:** `Linux, macOS, Windows (WSL/Git Bash)`

---

## The Instrument *(What It Is)*

Standard Unix CLI utilities (`grep`, `awk`, `sort`, `uniq`, `cut`) provide unmatched speed for immediate triage and hunting across gigabytes of authentication, web server, and syslog records during an active incident when a full SIEM is unavailable or disconnected.

---

## The Cases *(Battle-Tested One-Liners)*

### Case 1 — Linux Authentication (`/var/log/auth.log`)
```bash
# Top source IPs attempting failed SSH logins (Brute-force detection)
grep "Failed password" /var/log/auth.log | awk '{print $(NF-3)}' | sort | uniq -c | sort -rn | head -20

# Successful logins from unexpected IPs
grep "Accepted " /var/log/auth.log | awk '{print $9, $11}' | sort | uniq

# Sudo commands executed by users
grep "sudo:" /var/log/auth.log | grep -v "session\|opened\|closed"

# Newly created user accounts
grep -E "useradd|adduser|new user" /var/log/auth.log
```

### Case 2 — Web Server Access Logs (Apache / Nginx)
```bash
# Top 20 client IPs by total request volume (potential scanners / DoS)
awk '{print $1}' /var/log/apache2/access.log | sort | uniq -c | sort -rn | head -20

# Detect HTTP POST requests targeting PHP files (potential webshell interaction)
grep "POST.*\.php" /var/log/nginx/access.log

# Scan for common SQLi patterns in URL queries
grep -iE "(union|select|insert|drop|exec|waitfor|sleep)" /var/log/apache2/access.log
```

### Case 3 — Windows PowerShell Logs
```bash
# Search triage folder for base64 encoded PowerShell executions
grep -riE "(EncodedCommand|-enc |FromBase64String)" /path/to/logs/
```

---

## D3FEND / MITRE Mapping

| Framework | Reference | Capability |
|---|---|---|
| **D3FEND** | `d3f:LogAnalysis` | Operating System Event Analysis |
| **Counters ATT&CK** | `T1110`, `T1078`, `T1505.003` | Brute Force, Valid Accounts, Web Shells |

---

## Related Notes

- [[Blue/Instruments/chainsaw|Chainsaw]]
- [[Blue/Instruments/splunk|Splunk SPL Queries]]

↩ [[Blue/Instruments/readme|Blue: Instruments]] · [[Blue/Blue|Blue]] · [[Mind Palace]]
