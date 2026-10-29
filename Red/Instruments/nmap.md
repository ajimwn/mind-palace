---
type: tool
room: red
tool_name: "nmap"
version: "v7.9x"
platform: [Linux, Windows, macOS]
phase: [Reconnaissance, Scanning]
tags: [red/tool, red/recon, port-scanning, service-enumeration]
date_added: 2026-10-30
last_reviewed: 2026-10-30
---

# nmap

> [!quote]
> *"It is a capital mistake to theorize before one has data."*
> — *A Scandal in Bohemia*

**Room:** Red — Instruments | **Phase(s):** `Reconnaissance, Scanning` | **Platform:** `Linux, Windows, macOS`

---

## The Instrument *(What It Is)*

Network Mapper (`nmap`) is the foundational open-source network scanner for host discovery, port scanning, service version detection, OS fingerprinting, and automated vulnerability checks via the Nmap Scripting Engine (NSE).

- **Author / Origin:** Gordon "Fyodor" Lyon
- **License:** Nmap Public Source License (NPSL)
- **Install:** `sudo apt install nmap`

---

## The Method *(Core Usage)*

### Basic Syntax
```bash
nmap [Scan Type...] [Options] {target specification}
```

### Key Flags

| Flag | Meaning |
|------|---------|
| `-sS` | TCP SYN Stealth Scan (requires root / default for privileged users) |
| `-sT` | TCP Connect Scan (completes 3-way handshake; unprivileged) |
| `-sU` | UDP Port Scan |
| `-sV` | Probe open ports to determine service and version info |
| `-sC` | Run default NSE scripts equivalent to `--script=default` |
| `-A` | Aggressive scan: enables OS detection, version detection, scripts, traceroute |
| `-p-` | Scan all 65,535 ports |
| `-p <list>` | Scan specified ports (e.g. `-p 22,80,443,8080`) |
| `-Pn` | Treat all hosts as online — skip host discovery / ICMP ping |
| `-n` | Disable reverse DNS resolution for speed |
| `--min-rate <num>` | Send packets at minimum rate of `<num>` packets/sec |
| `-oA <basename>` | Output results in all three major formats (.nmap, .gnmap, .xml) |
| `--script <name>` | Execute specific NSE scripts or script categories |

---

## The Cases *(Common Use-Patterns)*

### Case 1 — Fast Full-Port Discovery
```bash
# Scan all 65,535 ports at high speed, skipping DNS and ping
sudo nmap -sS -p- --min-rate 5000 -Pn -n -oA recon/nmap-all 10.10.10.50
```

### Case 2 — Detailed Service & Script Enumeration
```bash
# Detailed investigation of discovered open ports
nmap -sC -sV -p 22,80,443,445,8080 -oA recon/nmap-detail 10.10.10.50
```

### Case 3 — NSE Vulnerability & Configuration Checks
```bash
# Check target for known vulnerabilities
nmap --script vuln -p 80,443 10.10.10.50

# Enumerate SMB shares and security
nmap --script "smb-enum-shares,smb-vuln-*" -p 139,445 10.10.10.50

# HTTP directory enumeration
nmap --script http-enum -p 80,443 10.10.10.50
```

### Case 4 — Subnet Host Discovery (Ping Sweep)
```bash
# Find live IPs in /24 subnet without scanning ports
nmap -sn 10.10.10.0/24 -oG - | grep Up | awk '{print $2}'
```

---

## MITRE ATT&CK Mapping

| Technique ID | Technique Name |
|-------------|----------------|
| `T1046` | Network Service Discovery |
| `T1595.001` | Active Scanning: Scanning IP Blocks |
| `T1595.002` | Active Scanning: Vulnerability Scanning |

---

## Gotchas & Caveats

> [!warning] Common Mistakes
> - **Omission of `-Pn`:** Many firewalls drop ICMP ping echo requests. Without `-Pn`, Nmap may mark an active host as down and terminate the scan.
> - **Default Top 1000 Ports:** Standard `nmap <target>` scans only the top 1000 TCP ports, missing services running on non-standard ports (e.g. 8080, 8443, 9001). Always run `-p-` for thorough pentests.

---

## OPSEC Notes

- SYN and Connect sweeps generate high packet velocity easily flagged by network firewalls and SIEM alarms.
- Use timing templates (`-T2` or `-T1`) and packet fragmentation (`-f`) when testing detection capabilities or stealth requirements.

---

## Related Notes

- [[Red/Monographs/TA0043 - Reconnaissance|TA0043 — Reconnaissance]]
- [[Red/Monographs/TA0007 - Discovery|TA0007 — Discovery]]
- [[Red/Instruments/tcpdump|tcpdump]]

↩ [[Red/Instruments/readme|Red: Instruments]] · [[Red/Red|Red]] · [[Mind Palace]]
