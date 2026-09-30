---
type: casefile
case: "{{case_name}}"
platform: "{{platform}}"  # THM / HTB / PG / VH / CTF
category: "{{category}}"  # Linux / Windows / Active Directory / Web / Cloud
difficulty: "{{difficulty}}"  # Easy / Medium / Hard / Insane
os: "{{os}}"
ip: "{{target_ip}}"
date: {{date}}
status: in-progress  # in-progress / rooted / completed
tags: [casebook, "{{platform_slug}}", "{{category_slug}}"]
---

# {{platform}} — {{case_name}}

> [!quote]
> *"Come, Watson, come! The game is afoot."*
> — *The Adventure of the Abbey Grange*

**Platform:** `{{platform}}` | **Difficulty:** `{{difficulty}}` | **Category:** `{{category}}` | **OS:** `{{os}}`
**IP:** `{{target_ip}}` | **Date:** `{{date}}`

---

## The Scene *(Machine Summary)*

Brief description of the machine's theme, technology stack, and what kind of challenge it presents. What does the name suggest?

---

## Phase 1 — Reconnaissance

> *"You have been in Afghanistan, I perceive." — observe before you theorise.*

### Port Scan
```bash
# Initial sweep
nmap -sS -p- --min-rate 5000 -oA recon/nmap-all {{target_ip}}

# Detailed scan on open ports
nmap -sC -sV -p <ports> -oA recon/nmap-detail {{target_ip}}
```

**Open Ports:**

| Port | Service | Version | Notes |
|------|---------|---------|-------|
| 22 | SSH | OpenSSH X.X | |
| 80 | HTTP | Apache/nginx | |

### Web Enumeration (if applicable)
```bash
gobuster dir -u http://{{target_ip}} -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt -o recon/gobuster.txt
ffuf -u http://{{target_ip}}/FUZZ -w /usr/share/seclists/Discovery/Web-Content/common.txt -mc 200,301,302
```

**Findings:**
-

---

## Phase 2 — Foothold

> *"When you have eliminated the impossible…" — what attack surface remains?*

### Vulnerability / Entry Point

**Identified:** (e.g. CVE-20XX-XXXX, SQL Injection, default credentials)

**MITRE ATT&CK:** `T{{id}}` — [[TA00XX - Tactic Name]]

```bash
# Commands used to gain initial shell
# ...

```

**Outcome:** Shell as user `___` on `{{target_ip}}`

---

## Phase 3 — Privilege Escalation

> *"The game is not over." — the investigation deepens.*

### Enumeration
```bash
# Automated
curl -sL https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh | sh
# OR: .\winpeas.exe

# Key findings:
```

**Key Findings:**
-

### Exploitation
```bash
# Privesc method (e.g. sudo -l → GTFOBin, SUID, weak service)
# ...

```

**Outcome:** Root / SYSTEM shell ✅

---

## Phase 4 — Post-Exploitation / Flags

```bash
# User flag
cat /home/user/user.txt

# Root flag
cat /root/root.txt
```

| Flag | Value |
|------|-------|
| user.txt | `[REDACTED — private vault only]` |
| root.txt | `[REDACTED — private vault only]` |

---

## The Deduction *(Attack Chain Summary)*

```
Recon → [Port/Service] → [Entry Point] → [Shell as X] → [PrivEsc Method] → Root
```

**MITRE ATT&CK Techniques Used:**

| Technique | ID | Phase |
|-----------|-----|-------|
| | T1XXX | Reconnaissance |
| | T1XXX | Initial Access |
| | T1XXX | Privilege Escalation |

---

## What Watson Missed *(Dead Ends)*

- Dead end 1 — why it didn't work
- Rabbit hole 2 — wasted time here because…

---

## Lessons Learned

> [!note] Key Takeaways
> - Most important lesson from this machine
> - Technique to remember: [[link to relevant TTP note]]
> - Tool discovered: [[link to tool note]]

---

## Countermeasures *(Blue Perspective)*

How would a defender detect or prevent the attack path used?

| Phase | Countermeasure | D3FEND |
|-------|---------------|--------|
| Initial Access | | |
| PrivEsc | | |

---

## References

- Machine page: {{platform_url}}
- CVE: {{cve_link}}
- Related notes: [[]]

↩ [[Casebook/Casebook]]
