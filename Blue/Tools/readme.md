---
type: section
room: blue
tags: [blue/tools]
---

# Blue: Instruments

> [!quote]
> *"I have a collection of some seventy-five perfumes, and the cases are often very instructive."*
> — *The Hound of the Baskervilles*

Analysis tools, queries, and command references for detection, investigation, and response.

---

## Inventory by Capability

### Log Analysis & SIEM
| Tool | Role | Note |
|------|------|------|
| `chainsaw` | Rapid Windows EVTX hunting | [[chainsaw]] |
| `hayabusa` | Windows timeline generator | [[hayabusa]] |
| `evtx_dump` | Convert EVTX to JSON | [[evtx_dump]] |
| Splunk | SIEM / SPL queries | [[Splunk Queries]] |
| Elastic/Kibana | KQL queries & dashboards | [[Elastic KQL]] |
| Microsoft Sentinel | KQL + UEBA | [[Sentinel KQL]] |

### Endpoint & Memory Forensics
| Tool | Role | Note |
|------|------|------|
| `volatility3` | Memory analysis | [[volatility]] |
| `velociraptor` | Live endpoint forensics | [[velociraptor]] |
| `autoruns` | Persistence enumeration | [[autoruns]] |
| `procmon` / `procexp` | Live process analysis | [[sysinternals]] |
| `FTK Imager` | Disk & memory acquisition | [[ftk-imager]] |
| `KAPE` | Artifact collection | [[kape]] |

### Network Analysis
| Tool | Role | Note |
|------|------|------|
| `wireshark` / `tshark` | Packet capture & analysis | [[wireshark]] |
| `zeek` | Network traffic analysis | [[zeek]] |
| `suricata` | IDS / network signatures | [[suricata]] |
| `NetworkMiner` | PCAP artifact extraction | [[networkminer]] |
| `nfdump` / NetFlow | Flow-based analysis | [[netflow]] |

### Malware Analysis
| Tool | Role | Note |
|------|------|------|
| `yara` | Pattern matching / IOC scanning | [[yara]] |
| `capa` | Malware capability detection | [[capa]] |
| `exiftool` | PE metadata extraction | [[exiftool]] |
| `die` / `Detect-It-Easy` | Packer/compiler identification | [[die]] |
| `strings` | String extraction | [[strings]] |
| `floss` | FLARE string deobfuscator | [[floss]] |

### Threat Intelligence
| Tool | Role | Note |
|------|------|------|
| VirusTotal | Hash/URL/IP reputation | [[virustotal]] |
| MISP | Threat intel sharing | [[misp]] |
| OpenCTI | CTI platform | [[opencti]] |
| `ioc-finder` | IOC extraction from text | [[ioc-finder]] |

### Detection Engineering
| Tool | Role | Note |
|------|------|------|
| Sigma | Detection rule standard | [[Sigma Rules]] |
| `sigma-cli` | Rule conversion & testing | [[sigma-cli]] |
| `uncoder.io` | Sigma → SIEM translation | [[uncoder]] |
| `atomic-red-team` | Detection validation | [[atomic-red-team]] |

---

## Command Cheatsheets

### Windows Live Response (PowerShell)
```powershell
# Who is logged in right now?
query user

# Active network connections
netstat -anob | Select-String -Pattern "ESTABLISHED"

# Recently created files (last 24h)
Get-ChildItem C:\ -Recurse -File | Where-Object { $_.CreationTime -gt (Get-Date).AddHours(-24) }

# Scheduled tasks (look for unusual paths)
schtasks /query /fo LIST /v | findstr /i "task\|run\|status"

# Services with non-standard paths
Get-WmiObject win32_service | Where-Object { $_.PathName -notlike "C:\Windows\*" } | Select-Object Name, PathName, State

# Autorun locations
reg query HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
reg query HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Run

# Recent PowerShell history
Get-Content (Get-PSReadlineOption).HistorySavePath
```

### Linux Live Response
```bash
# Logged in users
who; last | head -20

# Running processes
ps auxf

# Active network connections
ss -tulpn
netstat -tulpn

# Recently modified files
find / -mmin -60 -type f 2>/dev/null | grep -v /proc | grep -v /sys

# Cron jobs across all users
for user in $(cut -f1 -d: /etc/passwd); do echo "=== $user ==="; crontab -u $user -l 2>/dev/null; done

# SUID files
find / -perm -4000 -type f 2>/dev/null

# Listening services → map to process
ss -tulpn | awk '{print $5, $7}'
```

### Splunk SPL Essentials
```
# Top source IPs for failed auth
index=windows EventCode=4625
| stats count by src_ip, user
| sort -count

# Encoded PowerShell
index=windows EventCode=4688
| search CommandLine="*-enc*" OR CommandLine="*EncodedCommand*"
| table _time, ComputerName, user, CommandLine

# New scheduled tasks
index=windows EventCode=4698
| table _time, ComputerName, SubjectUserName, TaskName, TaskContent

# Beaconing detection (regular interval connections)
index=network
| bucket _time span=1m
| stats count by src_ip, dest_ip, dest_port, _time
| eventstats stdev(count) as stdev, avg(count) as avg by src_ip, dest_ip
| where stdev < 1 AND count > 10
```

---

## Conventions

- **One note per tool**, named after the tool
- **Template:** [[Templates/Blue - Detection Note]] for detection-focused tools
- **Tags:** `#blue/tool` + `#blue/<capability>` (e.g. `#blue/forensics`, `#blue/siem`)

↩ [[Blue]] · [[Mind Palace]]