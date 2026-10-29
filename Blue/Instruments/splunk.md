---
type: tool
room: blue
tool_name: "splunk"
version: "Splunk Enterprise / Cloud"
platform: [Linux, Windows]
phase: [Detection, SIEM, Threat Hunting]
tags: [blue/tool, blue/siem, splunk, spl, hunting-queries]
date_added: 2026-10-30
last_reviewed: 2026-10-30
---

# Splunk SPL (Hunting Queries)

> [!quote]
> *"It has long been an axiom of mine that the little things are infinitely the most important."*
> — *A Case of Identity*

**Room:** Blue — Instruments | **Role:** SIEM Search Processing Language (SPL) for Threat Detection | **Platform:** `Enterprise SIEM`

---

## The Instrument *(What It Is)*

Splunk Search Processing Language (SPL) is the standard query syntax used by Security Operations Centers (SOC) and threat hunters to correlate log streams across Windows endpoints, Active Directory, network firewalls, and cloud environments.

---

## The Method *(Core Usage)*

### Essential Hunting Queries

#### 1. Brute-Force & Failed Logins (Event 4625)
```splunk
index=windows EventCode=4625
| stats count by src_ip, user, ComputerName
| where count > 10
| sort -count
```

#### 2. Suspicious Process Execution & LOLBins (Event 4688 / Sysmon 1)
```splunk
index=windows (EventCode=4688 OR source="WinEventLog:Microsoft-Windows-Sysmon/Operational" EventCode=1)
| table _time, ComputerName, user, CommandLine
| where match(CommandLine, "(?i)(powershell|cmd|wscript|cscript|mshta|certutil|rundll32)")
```

#### 3. Encoded PowerShell Detection
```splunk
index=windows EventCode=4688
| where match(CommandLine, "(?i)(-enc|-EncodedCommand|frombase64)")
| table _time, host, user, CommandLine
```

#### 4. Lateral Movement over SMB (Event 4624 Logon Type 3)
```splunk
index=windows EventCode=4624 Logon_Type=3
| stats count by src_ip, user, ComputerName
| sort -count
```

#### 5. Malicious Service Installation (PsExec / Event 7045)
```splunk
index=windows EventCode=7045
| where match(ServiceFileName, "(?i)(PSEXE|ADMIN\$|psexesvc)")
| table _time, ComputerName, ServiceName, ServiceFileName
```

#### 6. DNS Tunneling & Long Query Length
```splunk
index=network sourcetype=dns
| stats count by query
| where len(query) > 50
| sort -count
```

---

## D3FEND / MITRE Mapping

| Framework | Reference | Capability |
|---|---|---|
| **D3FEND** | `d3f:LogAnalysis` | SIEM Log Analysis |
| **Counters ATT&CK** | `T1110`, `T1059.001`, `T1021` | Brute Force, PowerShell, Lateral Movement |

---

## Related Notes

- [[Blue/Instruments/chainsaw|Chainsaw]]
- [[Blue/Instruments/log-hunting|log-hunting (grep/awk)]]
- [[Blue/Monographs/IR - Detect and Analyse|IR — Detect and Analyse]]

↩ [[Blue/Instruments/readme|Blue: Instruments]] · [[Blue/Blue|Blue]] · [[Mind Palace]]
