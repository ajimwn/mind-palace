---
type: section
room: blue
tags: [blue/notes]
---

# Blue: Monographs

> [!quote]
> *"You see, but you do not observe. The distinction is clear."*
> — *A Scandal in Bohemia*

Detection engineering, incident response, and defensive tradecraft mapped to **MITRE D3FEND** and the **NIST IR Lifecycle**. For every Red tactic, Scotland Yard has an answer. Each note here is that answer.

---

## The NIST IR Lifecycle

```
┌─────────────┐     ┌─────────────────┐     ┌──────────────┐     ┌──────────────┐     ┌─────────────┐
│  PREPARE    │────▶│    DETECT &     │────▶│   CONTAIN    │────▶│  ERADICATE   │────▶│   RECOVER   │
│             │     │    ANALYSE      │     │              │     │              │     │             │
│ Controls    │     │ SIEM, Logs, EDR │     │ Isolate host │     │ Remove IOCs  │     │ Restore ops │
│ Playbooks   │     │ Threat hunting  │     │ Block IOC    │     │ Patch vuln   │     │ Lessons     │
└─────────────┘     └─────────────────┘     └──────────────┘     └──────────────┘     └─────────────┘
```

---

## MITRE D3FEND Capability Hubs

| D3FEND Category | Sub-capability | Note |
|-----------------|----------------|------|
| **Harden** | [[D3F - Application Hardening]] | Reduce attack surface |
| **Harden** | [[D3F - Credential Hardening]] | MFA, PAM, privilege reduction |
| **Harden** | [[D3F - Message Hardening]] | Email filtering, SPF/DKIM/DMARC |
| **Harden** | [[D3F - Network Hardening]] | Segmentation, firewall, ACLs |
| **Harden** | [[D3F - Platform Hardening]] | OS hardening, CIS benchmarks |
| **Detect** | [[D3F - File Analysis]] | Hash validation, YARA scanning |
| **Detect** | [[D3F - Identifier Analysis]] | IOC enrichment, threat intel |
| **Detect** | [[D3F - Message Analysis]] | Phishing detection |
| **Detect** | [[D3F - Network Traffic Analysis]] | IDS/IPS, NDR, flow analysis |
| **Detect** | [[D3F - Process Analysis]] | EDR, behavioural detection |
| **Detect** | [[D3F - User Behaviour Analysis]] | UEBA, insider threat |
| **Isolate** | [[D3F - Network Isolation]] | Micro-segmentation, quarantine |
| **Isolate** | [[D3F - Execution Isolation]] | Sandboxing, app whitelisting |
| **Deceive** | [[D3F - Decoy Environments]] | Honeypots, honeyfiles |
| **Evict** | [[D3F - Credential Eviction]] | Force password resets, revoke tokens |
| **Evict** | [[D3F - Process Eviction]] | Kill malicious processes, remediate |

---

## NIST IR Phase Notes

| Phase | Note | Covers |
|-------|------|--------|
| 🛡️ Prepare | [[IR - Prepare]] | Asset inventory, playbooks, detection rules |
| 🔍 Detect & Analyse | [[IR - Detect and Analyse]] | Log triage, SIEM, IOC pivot, malware analysis |
| 🚧 Contain | [[IR - Contain]] | Host isolation, account lockout, block rules |
| 🧹 Eradicate | [[IR - Eradicate]] | Remove persistence, patch, harden |
| 🔄 Recover | [[IR - Recover]] | Restore, validate, return to ops |
| 📝 Post-Incident | [[IR - Lessons Learned]] | Timeline, RCA, purple team findings |

---

## ATT&CK → D3FEND Mirror

> Every Red tactic has a Blue answer here.

| ATT&CK Tactic (Red) | D3FEND / Blue Counter |
|--------------------|-----------------------|
| TA0043 Reconnaissance | [[D3F - Network Traffic Analysis]], [[D3F - Identifier Analysis]] |
| TA0001 Initial Access | [[D3F - Message Hardening]], [[D3F - Application Hardening]], [[D3F - Network Traffic Analysis]] |
| TA0002 Execution | [[D3F - Process Analysis]], [[D3F - Execution Isolation]] |
| TA0003 Persistence | [[D3F - File Analysis]], [[D3F - Process Analysis]] |
| TA0004 Privilege Escalation | [[D3F - Process Analysis]], [[D3F - Credential Hardening]] |
| TA0005 Defense Evasion | [[D3F - Process Analysis]], [[D3F - File Analysis]] |
| TA0006 Credential Access | [[D3F - Credential Hardening]], [[D3F - User Behaviour Analysis]] |
| TA0008 Lateral Movement | [[D3F - Network Isolation]], [[D3F - Network Traffic Analysis]] |
| TA0011 C2 | [[D3F - Network Traffic Analysis]], [[D3F - Network Isolation]] |
| TA0010 Exfiltration | [[D3F - Network Traffic Analysis]], [[D3F - File Analysis]] |

---

## Conventions

- **IR Phase notes:** `IR - <Phase>.md` — follow [[Templates/Blue - IR Playbook]]
- **D3FEND notes:** `D3F - <Capability>.md` — follow [[Templates/Blue - Detection Note]]
- **Tags:** `#blue/<phase>` e.g. `#blue/detect`, `#blue/forensics`, `#blue/ir`
- **Event IDs** always cited alongside detections

↩ [[Blue]] · [[Mind Palace]]