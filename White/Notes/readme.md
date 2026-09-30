---
type: section
room: white
tags: [white/notes]
---

# White: Monographs

> [!quote]
> *"Mycroft has his rails and he runs on them. His Pall Mall lodgings, the Diogenes Club — that is his orbit, and it never varies."*
> — *The Adventure of the Bruce-Partington Plans*

Governance, Risk, and Compliance. Where Mycroft operates — quietly, invisibly — ensuring that the entire enterprise runs within rules. This room maps the frameworks that give a security programme its backbone.

---

## Framework Hubs

### Governance & Compliance Frameworks
| Framework | Note | Scope |
|-----------|------|-------|
| ISO/IEC 27001 | [[GRC - ISO 27001]] | ISMS standard — controls + certification |
| NIST CSF 2.0 | [[GRC - NIST CSF]] | Govern → Identify → Protect → Detect → Respond → Recover |
| SOC 2 | [[GRC - SOC 2]] | Trust Service Criteria for service orgs |
| PCI DSS | [[GRC - PCI DSS]] | Payment card data security |
| GDPR | [[GRC - GDPR]] | EU personal data regulation |
| HIPAA | [[GRC - HIPAA]] | US healthcare data |
| CIS Controls v8 | [[GRC - CIS Controls]] | Prioritised technical controls |

### Risk Management
| Subject | Note |
|---------|------|
| Risk Assessment Process | [[Risk - Risk Assessment]] |
| Threat Modelling (STRIDE) | [[Risk - Threat Modelling]] |
| Asset Classification | [[Risk - Asset Classification]] |
| BIA & Recovery Objectives | [[Risk - BIA and RTO RPA]] |

### Audit & Assessment
| Subject | Note |
|---------|------|
| Penetration Test Report Writing | [[Audit - Pentest Report]] |
| Vulnerability Management | [[Audit - Vuln Management]] |
| Security Policy Writing | [[Audit - Policy Writing]] |
| CVSS Scoring | [[Audit - CVSS]] |

---

## NIST CSF 2.0 at a Glance

```
GOVERN ──────────────────────────────────────────────────────────────
  Establish cybersecurity strategy, roles, policy, supply chain risk

IDENTIFY ────────────────────────────────────────────────────────────
  Asset management, risk assessment, improvement planning

PROTECT ─────────────────────────────────────────────────────────────
  Identity management, awareness training, data security, hardening

DETECT ──────────────────────────────────────────────────────────────
  Continuous monitoring, anomaly detection, event logging

RESPOND ─────────────────────────────────────────────────────────────
  Incident response, communications, analysis, mitigation

RECOVER ─────────────────────────────────────────────────────────────
  Recovery planning, improvements, communications
```

---

## CIS Controls v8 — Top 5 (Highest ROI)

| Control | Description | Why It Matters |
|---------|-------------|----------------|
| CIS 1 | Inventory & Control of Enterprise Assets | You can't protect what you don't know exists |
| CIS 2 | Inventory & Control of Software Assets | Unknown software = unknown attack surface |
| CIS 4 | Secure Configuration of Enterprise Assets | Defaults are adversary-friendly |
| CIS 5 | Account Management | Reduce privileged account exposure |
| CIS 6 | Access Control Management | Least privilege prevents lateral movement |

---

## Conventions

- **Framework notes:** `GRC - <Framework>.md`
- **Risk notes:** `Risk - <Topic>.md`
- **Audit notes:** `Audit - <Topic>.md`
- **Tags:** `#white/<category>` e.g. `#white/grc`, `#white/risk`, `#white/audit`

↩ [[White]] · [[Mind Palace]]