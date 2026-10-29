---
type: tool
room: red
tool_name: "hashcat"
version: "v6.2.x"
platform: [Linux, Windows, macOS]
phase: [Credential Access]
tags: [red/tool, red/credential-access, hash-cracking, gpu-cracking]
date_added: 2026-10-30
last_reviewed: 2026-10-30
---

# hashcat

> [!quote]
> *"To a great mind, nothing is little."*
> — *A Study in Scarlet*

**Room:** Red — Instruments | **Phase(s):** `Credential Access` | **Platform:** `Linux, Windows, macOS`

---

## The Instrument *(What It Is)*

`Hashcat` is the world's fastest CPU/GPU-based password cracking utility. Supporting hundreds of hash algorithms, rule engines, mask attacks, and dictionary permutations, it is the primary instrument for offline recovery of cryptographic hashes (NTLM, NetNTLMv2, Kerberos tickets, UNIX shadow passwords, WPA handshakes).

- **Author / Origin:** Jens Steube (atom)
- **License:** MIT License
- **Install:** `sudo apt install hashcat`

---

## The Method *(Core Usage)*

### Basic Syntax
```bash
hashcat -m <hash_mode> -a <attack_mode> <hashes_file> [wordlist|mask] [options]
```

### Essential Hash Modes (`-m`)

| Mode | Hash Type | Context |
|:---:|---|---|
| `0` | MD5 | Legacy systems, web apps |
| `100` | SHA1 | Git objects, legacy web apps |
| `1000` | NTLM | Windows SAM database, LSASS dumps |
| `1800` | SHA-512 ($6$) | Modern Linux `/etc/shadow` |
| `3200` | bcrypt ($2a$, $2b$) | Modern web application logins |
| `5600` | NetNTLMv2 | Captured via Responder / LLMNR poisoning |
| `13100` | Kerberos 5 TGS-REP | Kerberoasting (Active Directory SPNs) |
| `18200` | Kerberos 5 AS-REP | ASREPRoasting (Do not require Pre-Auth) |
| `22000` | WPA-PBKDF2-PMKID/EAPOL | WiFi handshakes |

### Attack Modes (`-a`)
- `-a 0` — Straight / Dictionary Attack
- `-a 1` — Combination Attack (words from two dictionaries concatenated)
- `-a 3` — Mask / Brute-force Attack (patterns e.g. `?u?l?l?d?d?d`)

---

## The Cases *(Common Use-Patterns)*

### Case 1 — Cracking NTLM Hashes from SAM / NTDS.dit
```bash
# Dictionary attack against extracted NTLM hashes
hashcat -m 1000 ntlm_hashes.txt /usr/share/wordlists/rockyou.txt
```

### Case 2 — Cracking Responder NTLMv2 Captures
```bash
# Crack NetNTLMv2 hash using RockYou and the best64 rule mutations
hashcat -m 5600 responder_capture.txt /usr/share/wordlists/rockyou.txt \
  -r /usr/share/hashcat/rules/best64.rule
```

### Case 3 — Kerberoasting (TGS-REP) Cracking
```bash
# Crack Active Directory Kerberoasted service ticket
hashcat -m 13100 kerb_tickets.txt /usr/share/wordlists/rockyou.txt
```

### Case 4 — Mask Attack
```bash
# Mask: 1 Capital letter, 3 lowercase, 4 digits (e.g. Summer2024)
hashcat -m 1000 ntlm.txt -a 3 ?u?l?l?l?d?d?d?d
```

### Case 5 — Displaying Already Cracked Hashes
```bash
# Show plaintext matches from potfile
hashcat -m 1000 ntlm.txt --show
```

---

## MITRE ATT&CK Mapping

| Technique ID | Technique Name |
|-------------|----------------|
| `T1110.002` | Brute Force: Password Cracking |
| `T1558.003` | Steal or Forge Kerberos Tickets: Kerberoasting |
| `T1558.004` | Steal or Forge Kerberos Tickets: AS-REP Roasting |

---

## Related Notes

- [[Red/Instruments/responder|Responder]]
- [[Red/Instruments/netexec|netexec]]
- [[Red/Monographs/TA0006 - Credential Access|TA0006 — Credential Access]]

↩ [[Red/Instruments/readme|Red: Instruments]] · [[Red/Red|Red]] · [[Mind Palace]]
