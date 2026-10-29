---
type: tool
room: red
tool_name: "linpeas"
version: "PEASS-ng latest"
platform: [Linux, Windows]
phase: [Privilege Escalation]
tags: [red/tool, red/privesc, enumeration, peass, suid, sudo]
date_added: 2026-10-30
last_reviewed: 2026-10-30
---

# linpeas / winpeas

> [!quote]
> *"Never trust to general impressions, my boy, but concentrate yourself upon details."*
> — *A Case of Identity*

**Room:** Red — Instruments | **Phase(s):** `Privilege Escalation` | **Platform:** `Linux, Windows`

---

## The Instrument *(What It Is)*

`PEASS-ng` (Privilege Escalation Awesome Scripts Suite - Next Generation) consists of `linpeas` (Linux) and `winpeas` (Windows). They are automated enumeration scripts that inspect the host system for privilege escalation vectors: misconfigured `sudo` permissions, vulnerable kernel versions, SUID/SGID binaries, cron jobs, stored credentials, registry misconfigurations, unquoted service paths, and Active Directory privileges.

- **Author / Origin:** Carlos Polop (@carlospolop)
- **License:** GNU GPL v3
- **Repository:** https://github.com/carlospolop/PEASS-ng

---

## The Method *(Core Usage)*

### Execution Flow
1. Transfer or stream the script directly to target memory.
2. Execute and output into a file or terminal pager (`less -r`).
3. Focus attention on **RED/YELLOW** highlights (99% probability of privilege escalation vector).

---

## The Cases *(Common Use-Patterns)*

### Case 1 — Linux In-Memory Execution
```bash
# Download and execute in memory via curl (leaves no file on disk)
curl -sL https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh | sh

# Save output locally to review with color
./linpeas.sh -a 2>/dev/null | tee /tmp/linpeas.txt
```

### Case 2 — Windows winpeas Execution
```powershell
# Run winpeas from PowerShell
.\winpeas.exe > winpeas.txt

# Run fast checks without waiting on heavy network sweeps
.\winpeas.exe quiet cmd fast > winpeas_fast.txt
```

---

## MITRE ATT&CK Mapping

| Technique ID | Technique Name |
|-------------|----------------|
| `T1082` | System Information Discovery |
| `T1069` | Permission Groups Discovery |
| `T1548` | Abuse Elevation Control Mechanism |

---

## Related Notes

- [[Red/Monographs/TA0004 - Privilege Escalation|TA0004 — Privilege Escalation]]

↩ [[Red/Instruments/readme|Red: Instruments]] · [[Red/Red|Red]] · [[Mind Palace]]
