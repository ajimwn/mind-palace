---
type: section
room: white
tags: [white/tools]
---

# White: Instruments

> [!quote]
> *"Mycroft's conclusions were as infallible as so many algebraic propositions."*

Templates, checklists, and reference tools for GRC practitioners.

---

## Inventory

### Assessment Tools
| Tool | Role | Note |
|------|------|------|
| `lynis` | Linux hardening audit | [[lynis]] |
| `CIS-CAT` | CIS benchmark scanner | [[cis-cat]] |
| OpenSCAP | SCAP compliance scanning | [[openscap]] |
| `nessus` / `openvas` | Vulnerability scanner | [[nessus]] |
| DefectDojo | Vulnerability management | [[defectdojo]] |

### Threat Modelling
| Tool | Role | Note |
|------|------|------|
| OWASP Threat Dragon | Threat model diagrams | [[threat-dragon]] |
| Microsoft STRIDE | STRIDE framework | [[stride]] |
| MITRE ATT&CK Navigator | TTP coverage heatmap | [[attack-navigator]] |

### Policy & Documentation
| Template | Note |
|----------|------|
| Acceptable Use Policy | [[Template - AUP]] |
| Incident Response Policy | [[Template - IR Policy]] |
| Vulnerability Disclosure Policy | [[Template - VDP]] |
| Pentest Report | [[Template - Pentest Report]] |
| Risk Register | [[Template - Risk Register]] |

---

## CVSS v3.1 Quick Scoring Guide

```
Base Score = f(AV, AC, PR, UI, S, C, I, A)

Attack Vector (AV):     Network(N) > Adjacent(A) > Local(L) > Physical(P)
Attack Complexity (AC): Low(L) > High(H)
Privileges Required (PR): None(N) > Low(L) > High(H)
User Interaction (UI):  None(N) > Required(R)
Scope (S):              Changed(C) > Unchanged(U)
CIA Impact:             High(H) > Low(L) > None(N)

Score Ranges:
  Critical: 9.0 – 10.0
  High:     7.0 – 8.9
  Medium:   4.0 – 6.9
  Low:      0.1 – 3.9
  None:     0.0
```

**Quick Reference:** [nvd.nist.gov/vuln-metrics/cvss](https://nvd.nist.gov/vuln-metrics/cvss)

---

## Risk Rating Matrix

```
         │  Low Impact │  Med Impact │  High Impact
─────────┼─────────────┼─────────────┼──────────────
High     │   MEDIUM    │    HIGH     │   CRITICAL
Likelihood─────────────┼─────────────┼──────────────
Medium   │    LOW      │   MEDIUM    │    HIGH
─────────┼─────────────┼─────────────┼──────────────
Low      │    INFO     │    LOW      │   MEDIUM
```

---

## Conventions

- **Templates:** `Template - <Name>.md`
- **Tags:** `#white/tool`, `#white/template`

↩ [[White]] · [[Mind Palace]]