---
type: section
room: blue
tags: [blue/monographs]
---

# Blue: Monographs

> [!quote]
> *"You have been in Afghanistan, I perceive." — deduce. Observe. Respond.*

Defensive studies mapped to **MITRE D3FEND** and the **NIST IR lifecycle**. Each hub covers detection methods, log sources, alert conditions, and response playbooks.

---

## IR Lifecycle — NIST SP 800-61

```
PREPARE ──► DETECT & ANALYSE ──► CONTAIN ──► ERADICATE ──► RECOVER ──► POST-INCIDENT
               │                     │            │             │              │
          IOC Hunting          Isolate host    Remove C2    Restore from    Lessons
          Log Analysis         Network seg     Clean artefacts  backups     Learned
          Alert triage         Credential      Patch vuln
                               reset
```

---

## Investigation Monographs

| Subject | Phase | Status |
|---------|-------|--------|
| [[IR - Detect and Analyse]] | Detect & Analyse | ✅ Ready |
| [[IOC Hunting & Threat Intel]] | Detect & Analyse | ✅ Ready |
| [[IR - Contain]] | Contain | 🔲 Planned |
| [[IR - Eradicate]] | Eradicate | 🔲 Planned |
| [[IR - Recover]] | Recover | 🔲 Planned |

---

## D3FEND Categories

| Category | Techniques | Mapped To |
|----------|-----------|-----------|
| Harden | Application, Credential, Message, Network, Platform Hardening | Protect |
| Detect | File, Identifier, Message, Network, Platform, Process Analysis | Detect |
| Isolate | Execution, Network Isolation | Contain |
| Deceive | Decoy Environment, Object, User | Detect + Confuse |
| Evict | Credential, Process, Object Eviction | Eradicate |
| Restore | Asset, Backup, Reconstitution | Recover |

---

## NIST 800-53 / CIS Controls Mapping

| CIS Control | Defensive Action | D3FEND |
|-------------|-----------------|--------|
| CIS 1 — Asset Inventory | Know your estate | Platform Monitoring |
| CIS 6 — Log Management | Centralise logs (SIEM) | Identifier Activity Analysis |
| CIS 7 — Email / Web Protections | Proxy + mail filter | Message Analysis |
| CIS 8 — Malware Defences | EDR + AV | File Analysis |
| CIS 13 — Network Monitoring | NDR, Zeek, NetFlow | Network Traffic Analysis |
| CIS 17 — IR Management | IR Plan + Playbooks | All |

---

## Conventions

- **IR playbooks:** `IR - <Phase>.md`
- **Detection guides:** `D3F - <Technique>.md`
- **Threat intel:** `TI - <Subject>.md`
- **Tags:** `#blue/<category>` e.g. `#blue/detection`, `#blue/ir`, `#blue/threat-intel`

↩ [[Blue]] · [[Mind Palace]]