---
type: section
room: red
tags: [red/monographs]
---

# Red: Monographs

> [!quote]
> *"It is a capital mistake to theorize before one has data. Insensibly one begins to twist facts to suit theories, instead of theories to suit facts."*
> — *A Scandal in Bohemia*

Studies of attack techniques mapped to the **MITRE ATT&CK** framework. Each tactic hub covers: techniques, tools, standard commands, OPSEC considerations, and what blue team detection looks like.

---

## The MITRE ATT&CK Tactic Chain

```
TA0043          TA0001          TA0002          TA0003          TA0004
Reconnaissance → Initial Access → Execution   → Persistence  → Privilege Escalation
                                                                      ↓
TA0007          TA0006          TA0008          TA0009          TA0005
Discovery     ← Credential     ← Lateral       ← Collection   ← Defense Evasion
                Access           Movement
                                                                      ↓
                                              TA0010          TA0011          TA0040
                                              Exfiltration ← C2           ← Impact
```

---

## Tactic Hubs

| Tactic ID | Name | Status |
|-----------|------|--------|
| [[TA0043 - Reconnaissance]] | Reconnaissance | ✅ Ready |
| [[TA0001 - Initial Access]] | Initial Access | ✅ Ready |
| [[TA0002 - Execution]] | Execution | ✅ Ready |
| [[TA0003 - Persistence]] | Persistence | ✅ Ready |
| [[TA0004 - Privilege Escalation]] | Privilege Escalation | ✅ Ready |
| [[TA0005 - Defense Evasion]] | Defense Evasion | ✅ Ready |
| [[TA0006 - Credential Access]] | Credential Access | ✅ Ready |
| [[TA0007 - Discovery]] | Discovery | ✅ Ready |
| [[TA0008 - Lateral Movement]] | Lateral Movement | ✅ Ready |
| [[TA0009 - Collection]] | Collection | ✅ Ready |
| [[TA0011 - Command and Control]] | Command and Control | ✅ Ready |
| [[TA0010 - Exfiltration]] | Exfiltration | ✅ Ready |
| [[TA0040 - Impact]] | Impact | ✅ Ready |

---

## Conventions

- **Tactic hubs:** `TA#### - Name.md` — one file per tactic, all tools and commands inside
- **Technique deep-dives:** `T#### - Name.md` — use [[Templates/Red - TTP Note]] for individual techniques
- **Tags:** `#red/<tactic_slug>` e.g. `#red/recon`, `#red/privesc`, `#red/lateral-movement`
- **Naming:** Follow ATT&CK IDs exactly so notes cross-link cleanly

---

## Quick Wins Index

> [!tip] Most Commonly Reached For
> These are the highest-ROI techniques in CTF and pentest contexts:
> - [[TA0043 - Reconnaissance]] — always the first move
> - [[TA0004 - Privilege Escalation]] — linpeas + sudo -l + SUID
> - [[TA0006 - Credential Access]] — Kerberoast, hash dump
> - [[TA0008 - Lateral Movement]] — Pass-the-Hash, PsExec, WinRM

↩ [[Red]] · [[Mind Palace]]