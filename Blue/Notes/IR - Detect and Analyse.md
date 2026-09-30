---
type: ir-phase
room: blue
phase: "Detect & Analyse"
nist_phase: "Detection and Analysis"
tags: [blue/detect, blue/analyse, blue/siem, blue/forensics]
date_updated: {{date}}
---

# IR Phase 2 — Detect & Analyse

> [!quote]
> *"You have been in a crime scene all night, Watson, and yet you notice nothing. I, however, observe everything."*

**NIST Phase:** Detection and Analysis | **D3FEND:** Process Analysis, Network Traffic Analysis, File Analysis

The first responder's discipline: cutting through the noise to find the signal. Before containing anything, Scotland Yard must first understand *what* they are dealing with and *how far it has spread*.

---

## Detection Sources

| Source | What It Catches | Tool / Query |
|--------|----------------|--------------|
| SIEM | Correlated multi-source events | Splunk, Elastic, Sentinel |
| EDR | Process, file, memory telemetry | CrowdStrike, Defender for Endpoint |
| IDS/IPS | Network signatures | Snort, Suricata |
| Firewall / NDR | Unusual outbound traffic | Zeek, Arkime |
| Windows Event Logs | Auth, process, registry changes | Event Viewer, Chainsaw |
| Sysmon | Enhanced process & network telemetry | Sysinternals Sysmon |
| Linux auditd | Syscall-level activity | auditd + ausearch |

---

## Windows Event ID Quick-Reference

### Authentication
| Event ID | Meaning | Red Flag |
|----------|---------|----------|
| 4624 | Successful logon | Logon Type 3 (network) from unexpected source |
| 4625 | Failed logon | Repeated failures → credential spray |
| 4648 | Explicit credential logon | RunAs / lateral movement indicator |
| 4672 | Special privilege assigned | Admin logon — note who and when |
| 4776 | Credential validation (NTLM) | NTLM on domain → possible hash relay |

### Process & Execution
| Event ID | Meaning | Red Flag |
|----------|---------|----------|
| 4688 | New process created | Unusual parent-child (e.g. Word → cmd) |
| 4689 | Process exited | Short-lived processes |
| 1 (Sysmon) | Process creation | Full command line — gold standard |
| 10 (Sysmon) | Process access | LSASS read → credential dumping |

### Object Access & Registry
| Event ID | Meaning | Red Flag |
|----------|---------|----------|
| 4663 | Object access | Access to sensitive files |
| 4698 | Scheduled task created | Persistence check |
| 4702 | Scheduled task modified | Existing task hijacked |
| 13 (Sysmon) | Registry value set | Autorun key modification |

### Network
| Event ID | Meaning | Red Flag |
|----------|---------|----------|
| 3 (Sysmon) | Network connection | Process making unexpected outbound |
| 5156 | WFP allowed connection | Baseline & hunt anomalies |
| 5158 | WFP blocked connection | Failed C2 connection attempts |

---

## Log Analysis Workflows

### Initial Triage (First 15 Minutes)

```powershell
# Windows — pull recent security events
Get-WinEvent -LogName Security -MaxEvents 1000 |
  Where-Object { $_.Id -in 4624,4625,4648,4688,4698 } |
  Select-Object TimeCreated, Id, Message |
  Export-Csv triage.csv

# Sysmon — process creation in last hour
Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" |
  Where-Object { $_.Id -eq 1 -and $_.TimeCreated -gt (Get-Date).AddHours(-1) } |
  Select-Object TimeCreated, Message
```

```bash
# Linux — auth failures and sudo usage
grep "Failed password" /var/log/auth.log | tail -100
grep "sudo:" /var/log/auth.log | tail -50

# Bash history of suspicious user
cat /home/<user>/.bash_history

# Active connections at time of incident (if live)
ss -tulpn
netstat -tulpn

# Recently modified files
find / -mmin -60 -type f 2>/dev/null | grep -v proc
```

### Chainsaw (Rapid Windows Log Hunting)
```bash
# Hunt for key attack techniques in Windows EVTX logs
chainsaw hunt /path/to/evtx/ --rules /opt/chainsaw/rules/ --mapping /opt/chainsaw/mapping_files/sigma-event-logs-all.yml

# Search for specific pattern
chainsaw search "mimikatz" /path/to/evtx/

# Timeline events from EVTX
chainsaw analyse /path/to/evtx/ --output timeline.csv
```

### Sigma Rule Quick-Search (Elastic / Splunk syntax)
```
# Suspicious process creation (Elastic KQL)
event.code: "4688" AND process.command_line: (*mimikatz* OR *sekurlsa* OR *lsadump*)

# LSASS access (Sysmon Event 10)
event.code: "10" AND winlog.event_data.TargetImage: "*lsass.exe"

# Encoded PowerShell
event.code: "4688" AND process.command_line: (*-enc* OR *-EncodedCommand* OR *FromBase64*)

# Lateral movement — PsExec pattern
event.code: "7045" AND winlog.event_data.ServiceFileName: (*ADMIN$* OR *psexesvc*)
```

---

## Memory Forensics

```bash
# Volatility 3 — Windows memory image
python3 vol.py -f memory.dmp windows.pslist         # process list
python3 vol.py -f memory.dmp windows.pstree         # process tree
python3 vol.py -f memory.dmp windows.netscan        # network connections
python3 vol.py -f memory.dmp windows.cmdline        # process command lines
python3 vol.py -f memory.dmp windows.dlllist --pid <pid>  # DLLs loaded
python3 vol.py -f memory.dmp windows.malfind        # injected code regions
python3 vol.py -f memory.dmp windows.dumpfiles --pid <pid>  # extract process

# Strings from suspicious region
python3 vol.py -f memory.dmp windows.strings --pid <pid> | grep -iE "(http|\\\\|pass|key)"
```

---

## Network Traffic Analysis

```bash
# Zeek connection log — large outbound transfers
zeek-cut id.orig_h id.resp_h id.resp_p orig_bytes resp_bytes < conn.log | \
  awk '$5 > 1000000' | sort -k5 -rn | head -20

# Capture and analyse with tshark
tshark -r capture.pcap -Y 'http.request.method == "POST"' -T fields \
  -e ip.src -e ip.dst -e http.host -e http.request.uri

# DNS exfiltration indicators
tshark -r capture.pcap -Y 'dns' -T fields -e dns.qry.name | \
  awk '{ print length($1), $1 }' | sort -rn | head -20  # long domain names = suspicious

# C2 beacon pattern (regular intervals)
zeek-cut id.orig_h id.resp_h ts < conn.log | \
  awk '{print $1, $2, int($3/60)}' | sort | uniq -c | sort -rn
```

---

## Malware Analysis (Static)

```bash
# File identification
file <sample>
strings <sample> | grep -E "(http|https|cmd|powershell|exec)"

# Hash and VirusTotal check
sha256sum <sample>
md5sum <sample>
# Submit hash to: https://www.virustotal.com

# PE analysis
objdump -x <sample>
exiftool <sample>
pecheck <sample>

# YARA scan
yara -r /path/to/rules/ <sample>

# Detect-it-Easy (DIE)
die <sample>
```

---

## D3FEND Countermeasures Applied Here

| D3FEND Technique | Implementation |
|-----------------|----------------|
| `D3-PA` Process Analysis | Sysmon Event ID 1, 10 + EDR behavioural rules |
| `D3-NTA` Network Traffic Analysis | Zeek + Suricata + flow baselining |
| `D3-FA` File Analysis | YARA scanning + hash reputation |
| `D3-UEBA` User Behaviour Analysis | Baseline logon times, alert on anomalies |

---

## Related Notes

- [[IR - Prepare]] ← Must be done first
- [[IR - Contain]] ← Next phase
- [[D3F - Network Traffic Analysis]]
- [[D3F - Process Analysis]]
- [[Blue - Instruments]]

↩ [[Blue - Notes]] · [[Blue]] · [[Mind Palace]]
