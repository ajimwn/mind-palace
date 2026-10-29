---
type: tool
room: blue
tool_name: "ioc-lookup"
version: "CLI Toolset"
platform: [Linux, macOS, Windows]
phase: [Threat Intelligence, Triage]
tags: [blue/tool, blue/threat-intel, ioc, virustotal, abuseipdb, shodan]
date_added: 2026-10-30
last_reviewed: 2026-10-30
---

# IOC Lookup (CLI & APIs)

> [!quote]
> *"It is a capital mistake to theorize before one has data."*
> — *A Scandal in Bohemia*

**Room:** Blue — Instruments | **Role:** Fast Indicator of Compromise (IOC) Enrichment & Verification | **Platform:** `Linux, macOS, Windows`

---

## The Instrument *(What It Is)*

Command-line utilities and API queries for instant enrichment of suspicious hashes, IP addresses, domains, and certificates discovered during an investigation.

---

## The Cases *(Quick CLI Queries)*

### Case 1 — VirusTotal File Hash Check
```bash
# Check file hash against VirusTotal database (vt CLI)
vt file <sha256_hash>
```

### Case 2 — MalwareBazaar (No API Key Required)
```bash
# Query hash details, malware family tags, and vendor intel
curl -s https://mb-api.abuse.ch/api/v1/ \
  -d "query=get_info&hash=<sha256_hash>" -X POST \
  | jq '.data[0].tags, .data[0].vendor_intel'
```

### Case 3 — AbuseIPDB Reputation Score
```bash
# Query IP address abuse confidence score
curl -G https://api.abuseipdb.com/api/v2/check \
  --data-urlencode "ipAddress=103.20.15.4" \
  -H "Key: <api_key>" \
  | jq '.data.abuseConfidenceScore, .data.countryCode, .data.usageType'
```

### Case 4 — Shodan Host & SSL Reconnaissance
```bash
# Lookup IP open services and vulnerability banners
shodan host <ip_address>

# Search Shodan by SSL certificate serial or fingerprint
shodan search "ssl:<cert_serial>"
```

### Case 5 — urlscan.io Domain Submissions
```bash
# Query recent automated scan results for suspicious domain
curl -s "https://urlscan.io/api/v1/search/?q=domain:<domain>&size=5" | jq '.results[].page.url'
```

---

## Related Notes

- [[Blue/Monographs/IOC Hunting & Threat Intel|IOC Hunting & Threat Intel]]
- [[Blue/Instruments/malware-analysis|Malware Analysis]]

↩ [[Blue/Instruments/readme|Blue: Instruments]] · [[Blue/Blue|Blue]] · [[Mind Palace]]
