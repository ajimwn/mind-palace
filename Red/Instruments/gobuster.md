---
type: tool
room: red
tool_name: "gobuster"
version: "v3.6.x"
platform: [Linux, Windows, macOS]
phase: [Reconnaissance, Scanning]
tags: [red/tool, red/web, dir-busting, vhost, dns-enumeration]
date_added: 2026-10-30
last_reviewed: 2026-10-30
---

# gobuster

> [!quote]
> *"It has long been an axiom of mine that the little things are infinitely the most important."*
> — *A Case of Identity*

**Room:** Red — Instruments | **Phase(s):** `Reconnaissance, Scanning` | **Platform:** `Linux, Windows, macOS`

---

## The Instrument *(What It Is)*

Gobuster is a high-performance directory, file, DNS subdomain, and Virtual Host (vhost) brute-forcing tool written in Go. Its concurrent execution makes it significantly faster than older Python- or Perl-based wordlist enumerators.

- **Author / Origin:** OJ Reeves (@The колосс) / Christian Mehlmauer (@firefart)
- **License:** Apache License 2.0
- **Install:** `sudo apt install gobuster` or `go install github.com/OJ/gobuster/v3@latest`

---

## The Method *(Core Usage)*

### Basic Modes
- `dir` — Directory/file brute-forcing mode
- `dns` — DNS subdomain enumeration mode
- `vhost` — Virtual host brute-forcing mode
- `fuzz` — Generic fuzzing mode

### Key Flags (`dir` mode)

| Flag | Meaning |
|------|---------|
| `-u <url>` | Target URL |
| `-w <wordlist>` | Path to wordlist |
| `-x <extensions>` | Comma-separated list of extensions to append (e.g. `php,html,txt`) |
| `-t <threads>` | Number of concurrent threads (default: 10) |
| `-o <file>` | Output file |
| `-k` | Skip TLS certificate verification |
| `-r` | Follow HTTP redirects |
| `-s <codes>` | Positive status codes to display (default: 200,204,301,302,307,401,403) |
| `-b <codes>` | Negative status codes to ignore |
| `--no-error` | Suppress error output |

---

## The Cases *(Common Use-Patterns)*

### Case 1 — Web Directory & File Enumeration
```bash
# Standard directory search with raft wordlist
gobuster dir -u http://10.10.10.50/ \
  -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt \
  -t 40 -o recon/gobuster-dir.txt

# File search with specific extensions
gobuster dir -u http://10.10.10.50/ \
  -w /usr/share/seclists/Discovery/Web-Content/raft-medium-files.txt \
  -x php,html,txt,bak,old,zip -t 40 -o recon/gobuster-files.txt
```

### Case 2 — Virtual Host (VHost) Enumeration
```bash
# Discover virtual hosts hosted on the same IP
gobuster vhost -u http://target.local \
  -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
  --append-domain -t 30 -o recon/vhosts.txt
```

### Case 3 — DNS Subdomain Discovery
```bash
# Brute-force subdomains against target authoritative nameserver
gobuster dns -d target.local \
  -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt \
  -r 10.10.10.50 -o recon/subdomains.txt
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
> - **Trailing Slashes:** Gobuster requests URLs exactly as built. If target endpoints redirect `/dir` to `/dir/` with a 301, enable `-r` to follow redirects or inspect both formats.
> - **Wildcard DNS / Catch-All VHosts:** When testing vhosts, servers often return 200 OK for every non-existent hostname. If every wordlist item returns 200, use `ffuf` with size/word filtering instead.

---

## Related Notes

- [[Red/Instruments/ffuf|ffuf]]
- [[Red/Monographs/TA0043 - Reconnaissance|TA0043 — Reconnaissance]]
- [[Red/Monographs/TA0001 - Initial Access|TA0001 — Initial Access]]

↩ [[Red/Instruments/readme|Red: Instruments]] · [[Red/Red|Red]] · [[Mind Palace]]
