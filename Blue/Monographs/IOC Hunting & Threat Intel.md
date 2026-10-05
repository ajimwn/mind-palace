---
type: monograph
room: blue
subject: "IOC Hunting & Threat Intelligence"
tags: [blue/threat-intel, blue/ioc, blue/hunting]
date_updated: {{date}}
---

# IOC Hunting & Threat Intelligence

> [!quote]
> *"There is nothing more practical than a good theory — and nothing more dangerous than bad intelligence."*

**Phase:** Detect & Analyse | **D3FEND:** `D3-IA` Identifier Analysis

The art of sourcing, validating, and actioning Indicators of Compromise — before or during an incident.

---

## Indicator Types (IOC Pyramid of Pain)

```
                    ▲  TTPs          ← Hardest for adversary to change
                   ▲▲  Tools
                  ▲▲▲  Network/Host Artifacts
                 ▲▲▲▲  Domain Names
                ▲▲▲▲▲  IP Addresses
               ▲▲▲▲▲▲  Hash Values  ← Easiest for adversary to change
```

> Focus hunting on TTPs and artifacts — hashes and IPs change daily.

| IOC Type               | Example                            | Volatility |
| ---------------------- | ---------------------------------- | ---------- |
| File hash (MD5/SHA256) | `d41d8cd98f00b204e9800998ecf8427e` | Minutes    |
| IP address             | `185.220.101.x`                    | Hours      |
| Domain name            | `evil.domain.com`                  | Days       |
| URL                    | `http://evil.com/payload.exe`      | Days       |
| Email address          | `phish@evil.com`                   | Weeks      |
| User-Agent string      | `Mozilla/5.0 (evil)`               | Weeks      |
| Registry key           | `HKCU\...\Run\update`              | Months     |
| TTP                    | T1566 Ph ishing → T1059 PowerShell | Long-term  |

---

## IOC Sources — Where to Find Them

### Free / Open Sources

| Source | URL | Best For |
|--------|-----|---------|
| **VirusTotal** | [virustotal.com](https://www.virustotal.com) | Hash, URL, IP, domain reputation |
| **MalwareBazaar** | [bazaar.abuse.ch](https://bazaar.abuse.ch) | Malware samples + hashes |
| **URLhaus** | [urlhaus.abuse.ch](https://urlhaus.abuse.ch) | Malicious URLs + payloads |
| **ThreatFox** | [threatfox.abuse.ch](https://threatfox.abuse.ch) | IOCs mapped to malware families |
| **MISP Feeds** | [misp-project.org/feeds](https://www.misp-project.org/feeds/) | Structured threat intel |
| **OTX AlienVault** | [otx.alienvault.com](https://otx.alienvault.com) | Community IOC pulses |
| **Shodan** | [shodan.io](https://shodan.io) | Internet-facing infrastructure |
| **Censys** | [censys.io](https://censys.io) | Certificate + banner fingerprinting |
| **GreyNoise** | [greynoise.io](https://greynoise.io) | Distinguish scanner vs targeted |
| **AbuseIPDB** | [abuseipdb.com](https://www.abuseipdb.com) | IP reputation + abuse reports |
| **PhishTank** | [phishtank.org](https://phishtank.org) | Phishing URL database |
| **URLScan.io** | [urlscan.io](https://urlscan.io) | URL analysis + screenshot |
| **Hybrid Analysis** | [hybrid-analysis.com](https://www.hybrid-analysis.com) | Sandbox analysis |
| **Any.run** | [any.run](https://any.run) | Interactive malware sandbox |
| **Joe Sandbox** | [joesandbox.com](https://www.joesandbox.com) | Automated sandbox |
| **LOLBAS** | [lolbas-project.github.io](https://lolbas-project.github.io) | Living off the land binaries |
| **GTFOBins** | [gtfobins.github.io](https://gtfobins.github.io) | Unix binary abuse |
| **Sigma Rules** | [github.com/SigmaHQ/sigma](https://github.com/SigmaHQ/sigma) | Detection rules |

### Threat Intel Feeds (Structured)

```bash
# Feodo Tracker — C2 IPs
curl https://feodotracker.abuse.ch/downloads/ipblocklist.txt

# URLhaus — malicious URLs
curl https://urlhaus.abuse.ch/downloads/text/

# MalwareBazaar — recent hashes (JSON)
curl -s https://mb-api.abuse.ch/api/v1/ -d "query=get_recent&selector=time" -X POST | jq '.data[].sha256_hash'

# OTX — pulse IOCs (requires API key)
curl -H "X-OTX-API-KEY: <your_key>" "https://otx.alienvault.com/api/v1/pulses/subscribed" | jq '.results[].indicators[]'
```

---

## IOC Lookup Commands

### Hash Lookup
```bash
# VirusTotal CLI (requires API key)
pip install vt-cli
vt file <sha256_hash>

# MalwareBazaar API (no key needed)
curl -s https://mb-api.abuse.ch/api/v1/ -d "query=get_info&hash=<sha256>" -X POST | jq '.data[0]'

# Identify hash type
echo "<hash>" | wc -c  # 33=MD5, 41=SHA1, 65=SHA256

# Hash a file locally
sha256sum <file>
md5sum <file>
Get-FileHash <file> -Algorithm SHA256   # PowerShell
```

### IP / Domain Lookup
```bash
# whois
whois <ip_or_domain>

# DNS resolution
dig <domain> +short
dig -x <ip> +short   # reverse DNS
nslookup <domain>

# Passive DNS (historical) — requires API keys
# pdns.passivetotal.org
# community.riskiq.com

# Shodan CLI
pip install shodan
shodan host <ip>
shodan search "org:<org_name>"

# AbuseIPDB
curl -G https://api.abuseipdb.com/api/v2/check \
  --data-urlencode "ipAddress=<ip>" \
  -d maxAgeInDays=90 \
  -H "Key: <api_key>" \
  -H "Accept: application/json" | jq '.data.abuseConfidenceScore'

# URLScan.io — scan a URL
curl -H "API-Key: <key>" -H "Content-Type: application/json" \
  -d '{"url":"http://suspect.domain.com","visibility":"public"}' \
  https://urlscan.io/api/v1/scan/
```

### Certificate Intelligence
```bash
# crt.sh — certificate transparency search
curl -s "https://crt.sh/?q=%25.<domain>&output=json" | jq '.[].name_value' | sort -u

# Censys
# Search certificates issued to a domain:
# censys.io/search?resource=certificates&q=parsed.names%3A<domain>
```

---

## IOC Pivoting — From One IOC to Many

```
Hash → VirusTotal → contacted_domains / contacted_ips / dropped_files
IP → Shodan → open_ports / banners / org / ASN / other_IPs_same_cert
Domain → crt.sh → subdomains → new_IPs
URL → urlscan.io → linked_domains / outbound_requests / screenshots
Email → Maltego / Hunter.io → org / related_domains
```

### Pivot Workflow Example
```bash
# 1. Start with a hash from an EDR alert
sha256=$(echo "<malware.exe_hash>")

# 2. Query VT for network IOCs
curl -H "x-apikey: <key>" "https://www.virustotal.com/api/v3/files/$sha256/contacted_domains" | jq '.data[].id'

# 3. For each domain, get IPs
dig <found_domain> +short

# 4. For each IP, check Shodan
shodan host <found_ip>

# 5. Check all IPs/domains in your SIEM
# Elastic:
# destination.ip: (<ip1> OR <ip2>) OR dns.question.name: (<domain1>)
```

---

## Threat Intel Platforms (Self-Hosted)

| Platform | Purpose | Setup |
|----------|---------|-------|
| **MISP** | IOC sharing, correlation | Docker: `docker run misp/misp` |
| **OpenCTI** | CTI platform, ATT&CK mapping | docker-compose available |
| **TheHive** | Case management + alert triage | docker-compose available |
| **Cortex** | Observable analysis (VirusTotal, Shodan) | Pairs with TheHive |

---

## Hunting Queries — Known Threat Actor IOCs

```bash
# SIEM — hunt for known bad IPs across all log sources
# Elastic KQL:
destination.ip: (185.220.101.0/24 OR 185.156.73.0/24) AND NOT source.ip: 10.0.0.0/8

# SIEM — hunt by domain
dns.question.name: *torproject.org OR dns.question.name: *duckdns.org

# SIEM — hunt for Cobalt Strike default malleable profiles
http.response.headers.content-type: "application/octet-stream" AND url.path: (/submit.php OR /updates.rss)
```

---

## Related Notes

- [[IR - Detect and Analyse]] ← Parent workflow
- [[D3F - Identifier Analysis]]
- [[Blue - Instruments]] ← Tools for IOC lookup

↩ [[Blue - Monographs]] · [[Blue]] · [[Mind Palace]]
