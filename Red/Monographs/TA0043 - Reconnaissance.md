---
type: tactic-hub
room: red
tactic: "Reconnaissance"
tactic_id: "TA0043"
tags: [red/recon, mitre/TA0043]
date_updated: {{date}}
---

# TA0043 — Reconnaissance

> [!quote]
> *"You have been in Afghanistan, I perceive."*
> — *A Study in Scarlet*

**MITRE Reference:** [TA0043](https://attack.mitre.org/tactics/TA0043/) | **Kill Chain Phase:** Pre-intrusion

The adversary is trying to gather information about the target. Before a single packet crosses the wire, Holmes walks the street, reads the newspapers, and asks the irregulars. Reconnaissance is intelligence collection — passive or active — that shapes every subsequent move.

---

## Phase Overview

| Sub-phase | Goal |
|-----------|------|
| Passive OSINT | Gather without touching the target |
| Active Scanning | Touch the target; enumerate live systems and services |
| Social Reconnaissance | Identify people, org structure, email patterns |

---

## Techniques in This Tactic

| Technique ID | Name | Priority |
|-------------|------|---------|
| [[T1595 - Active Scanning]] | Active Scanning | High |
| [[T1592 - Gather Victim Host Info]] | Gather Victim Host Info | High |
| [[T1589 - Gather Victim Identity Info]] | Gather Victim Identity Info | Medium |
| [[T1590 - Gather Victim Network Info]] | Gather Victim Network Info | High |
| [[T1591 - Gather Victim Org Info]] | Gather Victim Org Info | Medium |
| [[T1598 - Phishing for Information]] | Phishing for Information | Medium |
| [[T1597 - Search Closed Sources]] | Search Closed Sources | Low |
| [[T1596 - Search Open Technical Databases]] | Search Open Technical Databases | High |
| [[T1593 - Search Open Websites]] | Search Open Websites/Domains | Medium |
| [[T1594 - Search Victim-Owned Websites]] | Search Victim-Owned Websites | Medium |

---

## The Standard Kit *(Tools)*

| Tool | Role | Type | Command Stub |
|------|------|------|-------------|
| `nmap` | Port & service scan | Active | `nmap -sC -sV -oA recon/nmap <target>` |
| `masscan` | Rapid port sweep | Active | `masscan -p1-65535 <target> --rate=1000` |
| `amass` | Subdomain enumeration | Passive+Active | `amass enum -passive -d <domain>` |
| `subfinder` | Subdomain discovery | Passive | `subfinder -d <domain> -o subs.txt` |
| `theHarvester` | Email & subdomain harvest | Passive | `theHarvester -d <domain> -b all` |
| `shodan` CLI | Internet-facing exposure | Passive | `shodan host <ip>` |
| `whatweb` | Web tech fingerprinting | Active | `whatweb -a 3 <url>` |
| `whois` | Domain ownership & registrar | Passive | `whois <domain>` |
| `dnsx` | DNS resolution & bruting | Active | `dnsx -l subs.txt -a -resp` |
| `wafw00f` | WAF detection | Active | `wafw00f <url>` |

---

## Standard Recon Workflow

### Phase 1 — Passive (No Target Touch)

```bash
# --- Domain intelligence ---
whois <domain>
theHarvester -d <domain> -b google,bing,linkedin -f harvest.html

# --- Subdomain enumeration (passive) ---
subfinder -d <domain> -o subs-passive.txt
amass enum -passive -d <domain> -o subs-amass.txt
cat subs-passive.txt subs-amass.txt | sort -u > subs-all.txt

# --- Certificate transparency logs ---
curl -s "https://crt.sh/?q=%25.<domain>&output=json" | jq '.[].name_value' | sort -u

# --- Shodan lookup ---
shodan host <ip>
shodan search "org:<org_name>" --fields ip_str,port,hostnames
```

### Phase 2 — DNS Active Resolution

```bash
# Resolve discovered subdomains
dnsx -l subs-all.txt -a -resp -o resolved.txt

# DNS brute-force
dnsx -d <domain> -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt -o dns-brute.txt

# Zone transfer attempt (rarely works, worth trying)
dig axfr <domain> @<nameserver>
```

### Phase 3 — Port & Service Scan (Active)

```bash
# Fast sweep — all ports
nmap -sS -p- --min-rate 5000 -oA recon/nmap-allports <target>

# Service & script scan on discovered ports
ports=$(grep -oP '(?<=port=)\d+' recon/nmap-allports.xml | tr '\n' ',')
nmap -sC -sV -p$ports -oA recon/nmap-detailed <target>

# UDP top ports (slow, selective)
nmap -sU --top-ports 100 -oA recon/nmap-udp <target>

# Masscan for speed on large ranges
masscan -p0-65535 <cidr> --rate=10000 -oX recon/masscan.xml
```

### Phase 4 — Web Fingerprinting

```bash
# Detect technologies
whatweb -a 3 <url> | tee recon/whatweb.txt

# WAF detection
wafw00f <url>

# HTTP headers inspection
curl -sI <url>

# Favicon hash (Shodan technique)
python3 -c "
import requests, mmh3, codecs
r = requests.get('<url>/favicon.ico')
fhash = mmh3.hash(codecs.lookup('base64').encode(r.content)[0])
print(fhash)
"
```

---

## OPSEC Considerations

> [!warning] Stay Invisible
> - Passive OSINT leaves **zero footprint** — always exhaust it before going active.
> - `nmap -sS` requires root; without it, falls back to TCP connect (noisier).
> - Space out active scans; consecutive SYN sweeps trigger IDS/IPS threshold rules.
> - Use `--scan-delay` or `--max-rate` on noisy targets.
> - Rotate source IPs or use a VPN/proxy chain for active scanning in real engagements.

---

## What Blue Sees

| Activity | Log Source | Indicator |
|----------|-----------|-----------|
| Port scan | Firewall / IDS | Sequential port hits, RST floods |
| Subdomain brute | DNS server | High-volume NXDOMAIN responses |
| HTTP fingerprinting | Web server | Unusual User-Agent strings |
| `theHarvester` / scraping | — | No direct target footprint |

---

## Related Notes

- [[TA0001 - Initial Access]] ← Where recon feeds into
- [[Red - Instruments]] ← Full tool index

↩ [[Red - Notes]] · [[Red]] · [[Mind Palace]]
