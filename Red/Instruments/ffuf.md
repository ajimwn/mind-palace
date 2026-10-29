---
type: tool
room: red
tool_name: "ffuf"
version: "v2.x"
platform: [Linux, Windows, macOS]
phase: [Reconnaissance, Scanning, Exploitation]
tags: [red/tool, red/web, fuzzing, parameter-discovery]
date_added: 2026-10-30
last_reviewed: 2026-10-30
---

# ffuf

> [!quote]
> *"Data! Data! Data! I can't make bricks without clay."*
> — *The Adventure of the Copper Beeches*

**Room:** Red — Instruments | **Phase(s):** `Reconnaissance, Scanning` | **Platform:** `Linux, Windows, macOS`

---

## The Instrument *(What It Is)*

`ffuf` (Fastest FUZZer You Fool) is an extremely fast, modular web fuzzer written in Go. Its hallmark feature is the ability to place the `FUZZ` placeholder anywhere: URL paths, headers, POST data, HTTP methods, and query parameters. It provides unmatched filtering options by status code, response byte length, word count, line count, and regex.

- **Author / Origin:** Joonas Kuorilehto (@joohoi)
- **License:** MIT License
- **Install:** `sudo apt install ffuf` or `go install github.com/ffuf/ffuf/v2@latest`

---

## The Method *(Core Usage)*

### Basic Syntax
```bash
ffuf -u <url_with_FUZZ> -w <wordlist_path> [filtering_options]
```

### Key Flags

| Flag | Meaning |
|------|---------|
| `-u <url>` | Target URL containing the keyword `FUZZ` |
| `-w <path>` | Wordlist file path (or `file:KEYWORD` for multiple lists) |
| `-mc <codes>` | Match status codes (e.g. `-mc 200,301,302,403`) |
| `-fc <codes>` | Filter out status codes (e.g. `-fc 404`) |
| `-fs <sizes>` | Filter responses by HTTP response size in bytes |
| `-fw <words>` | Filter responses by word count |
| `-fl <lines>` | Filter responses by line count |
| `-X <method>` | HTTP method to use (GET, POST, PUT, etc.) |
| `-d <data>` | POST body data (e.g. `-d "user=admin&pass=FUZZ"`) |
| `-H <header>` | Custom HTTP Header (e.g. `-H "Host: FUZZ.target.local"`) |
| `-r` | Follow HTTP redirects |
| `-t <threads>` | Number of concurrent threads (default: 40) |
| `-o <file>` | Write output to JSON or CSV file |

---

## The Cases *(Common Use-Patterns)*

### Case 1 — Hidden Directory & Extension Fuzzing
```bash
# Discover directories and common web extensions
ffuf -u http://10.10.10.50/FUZZ \
  -w /usr/share/seclists/Discovery/Web-Content/raft-medium-files.txt \
  -e .php,.html,.txt,.bak -mc 200,301,302,403 -o recon/ffuf-dirs.json
```

### Case 2 — Virtual Host Fuzzing with Size Filtering
```bash
# Fuzz the Host header, filtering out the baseline false-positive size (e.g. 1542 bytes)
ffuf -u http://10.10.10.50/ \
  -H "Host: FUZZ.domain.local" \
  -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt \
  -fs 1542
```

### Case 3 — GET & POST Parameter Discovery
```bash
# Fuzz hidden GET parameters
ffuf -u "http://10.10.10.50/view.php?FUZZ=1" \
  -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt \
  -fs 0 -mc 200

# Fuzz JSON POST keys
ffuf -u http://10.10.10.50/api/endpoint -X POST \
  -H "Content-Type: application/json" \
  -d '{"FUZZ": "test"}' \
  -w /usr/share/seclists/Discovery/Web-Content/api/api-endpoints.txt
```

---

## MITRE ATT&CK Mapping

| Technique ID | Technique Name |
|-------------|----------------|
| `T1595.002` | Active Scanning: Vulnerability Scanning |
| `T1595.003` | Active Scanning: Wordlist Scanning |

---

## Gotchas & Caveats

> [!warning] Common Mistakes
> - **Calibration Before Blasting:** Always identify the default page size first! Run a dummy request (`curl -s http://target/nonexistent_12345 | wc -c`), note the byte length, and pass `-fs <bytes>` into `ffuf` to prevent thousands of false positives.

---

## Related Notes

- [[Red/Instruments/gobuster|gobuster]]
- [[Red/Monographs/TA0043 - Reconnaissance|TA0043 — Reconnaissance]]
- [[Red/Monographs/TA0001 - Initial Access|TA0001 — Initial Access]]

↩ [[Red/Instruments/readme|Red: Instruments]] · [[Red/Red|Red]] · [[Mind Palace]]
