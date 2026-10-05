---
type: monograph
room: white
subject: "NIST CSF 2.0"
framework: "NIST CSF"
tags: [white/grc, white/nist-csf]
date_updated: {{date}}
---

# GRC — NIST Cybersecurity Framework 2.0

> [!quote]
> *"It is not enough to have a good mind. The main thing is to use it well."*
> — adapted from Holmes

**Framework:** NIST CSF 2.0 (2024) | **Applies To:** Any organisation size/sector

The CSF 2.0 added **GOVERN** as the sixth function, making it the outer shell that all other functions operate within. It is the policy layer — the partner that ensures the security programme answers to leadership.

---

## The Six Functions

```
                    ┌─────────────────────────────────┐
                    │            GOVERN                │
                    │  (Strategy, Policy, Risk, Roles) │
                    └─────────────────────────────────┘
                                    │
         ┌──────────────────────────┼──────────────────────────┐
         ▼                          ▼                          ▼
    IDENTIFY                    PROTECT                    DETECT
  (Asset mgmt,              (IAM, Awareness,           (Monitoring,
   Risk assess,              Data security,             Anomaly
   Supply chain)             Hardening)                 detection)
         │                          │                          │
         └──────────────────────────┼──────────────────────────┘
                                    │
                          ┌─────────┴─────────┐
                          ▼                   ▼
                       RESPOND            RECOVER
                    (IR, Analysis,    (Restoration,
                     Mitigation)       Lessons)
```

---

## GOVERN (GV)

The new function in 2.0. Establishes and monitors the organisation's cybersecurity risk management strategy and policy.

| Category | Key Requirements |
|----------|-----------------|
| GV.OC — Organisational Context | Business objectives drive security decisions |
| GV.RM — Risk Management Strategy | Risk appetite, tolerance, and strategy documented |
| GV.RR — Roles & Responsibilities | CISO, security team, and business owners defined |
| GV.PO — Policy | Policies exist, are reviewed, and enforced |
| GV.OV — Oversight | Board/leadership review of cyber risk |
| GV.SC — Supply Chain Risk | Third-party risk management programme |

**Practical Implementation:**
```markdown
☐ Information Security Policy — approved by leadership, reviewed annually
☐ Risk Appetite Statement — quantified or qualitatively defined
☐ RACI Matrix — who is Responsible / Accountable / Consulted / Informed for security
☐ Third-party vendor questionnaires (SIG Lite, CAIQ)
☐ Cyber risk reported to board at least quarterly
```

---

## IDENTIFY (ID)

Know your assets, risks, and improvement opportunities.

| Category | Key Requirements |
|----------|-----------------|
| ID.AM — Asset Management | Inventory of hardware, software, data, and services |
| ID.RA — Risk Assessment | Identify threats, vulnerabilities, likelihood, impact |
| ID.IM — Improvement | Plan to improve security posture |

**Asset Inventory Minimum:**
```markdown
☐ Hardware asset inventory (CMDB) — updated quarterly
☐ Software asset inventory (authorised software list)
☐ Data classification scheme (Public / Internal / Confidential / Restricted)
☐ Data flow diagram for critical systems (PII, PCI data)
☐ Crown jewels identified (most critical systems/data)
```

**Risk Assessment Steps:**
```
1. Asset scoping    → What are we protecting?
2. Threat catalogue → What could go wrong? (MITRE ATT&CK, STRIDE)
3. Vulnerability ID → Where are the gaps? (pentest, vuln scan, audit)
4. Likelihood       → How probable is it? (1–5 scale)
5. Impact           → What is the consequence? (1–5 scale)
6. Risk score       → Likelihood × Impact
7. Treatment        → Accept / Mitigate / Transfer / Avoid
8. Residual risk    → Risk remaining after controls
9. Accept & review  → Owner signs off; review at next cycle
```

---

## PROTECT (PR)

Controls to limit or contain the impact of a cybersecurity event.

| Category | Key Requirements |
|----------|-----------------|
| PR.AA — Identity Management | MFA, least privilege, PAM |
| PR.AT — Awareness Training | Annual security training, phishing simulation |
| PR.DS — Data Security | Encryption at rest and transit, DLP, retention |
| PR.PS — Platform Security | Hardening, patching, secure configuration |
| PR.IR — Technology Infrastructure | Network segmentation, firewall, EDR |

**Patching SLAs (industry standard):**
```
Critical (CVSS 9.0+):  Patch within 24–72 hours
High     (CVSS 7.0+):  Patch within 7–14 days
Medium   (CVSS 4.0+):  Patch within 30 days
Low      (CVSS <4.0):  Patch within 90 days or next maintenance window
```

**Hardening Standards:**
- Windows: CIS Microsoft Windows Benchmark
- Linux: CIS Linux Benchmark / DISA STIG
- Cloud: CIS AWS/Azure/GCP Foundations Benchmark
- Containers: CIS Docker/Kubernetes Benchmark

---

## DETECT (DE)

Identify cybersecurity events in a timely manner.

| Category | Key Requirements |
|----------|-----------------|
| DE.CM — Continuous Monitoring | SIEM, EDR, NDR, vulnerability scanning |
| DE.AE — Adverse Event Analysis | Triage, correlation, alert quality |

**Detection Stack Minimum:**
```markdown
☐ Centralised logging (SIEM) — Windows events, Linux syslog, network flows
☐ Log retention: 90 days hot, 12 months cold (regulated: 12 months hot)
☐ EDR on all endpoints
☐ Vulnerability scanner runs monthly (continuous is better)
☐ Alert thresholds tuned — review false positive rate quarterly
☐ MTTD (Mean Time to Detect) — target < 24 hours for high alerts
```

**Key Log Sources to Collect:**
```
Windows:  Security (4624, 4625, 4688, 4698, 4720, 7045)
          Sysmon (1, 3, 7, 10, 11, 12, 13)
Linux:    auth.log, syslog, auditd
Network:  Firewall deny logs, DNS query logs, NetFlow/IPFIX
Web:      Access logs, WAF alerts
Email:    Anti-spam, DMARC reports
Cloud:    CloudTrail (AWS), Unified Audit Log (M365), Cloud Audit (GCP)
```

---

## RESPOND (RS)

Take action regarding a detected cybersecurity incident.

| Category | Key Requirements |
|----------|-----------------|
| RS.MA — Incident Management | IR plan, escalation paths |
| RS.AN — Incident Analysis | Root cause, scope, indicators |
| RS.CO — Incident Response Reporting | Notify stakeholders, regulators |
| RS.MI — Incident Mitigation | Contain and mitigate impact |

**IR Plan Must Include:**
```markdown
☐ Incident classification (P1–P4) with response SLAs
☐ Escalation contacts (CISO, Legal, PR, Exec, External IR retainer)
☐ Regulatory notification timelines (GDPR: 72 hours; PCI: immediately)
☐ Evidence preservation procedures (chain of custody)
☐ Communication templates (internal + external)
☐ War room / incident Slack channel procedure
```

**Notification Timelines by Regulation:**
| Regulation | Notification Window | To Whom |
|------------|---------------------|---------|
| GDPR | 72 hours | Supervisory Authority (e.g. ICO) |
| PCI DSS | Immediately | Card brands + acquirer |
| HIPAA | 60 days | HHS (breach >500 individuals) |
| NIS2 | 24h (early warning), 72h (notification) | National CSIRT |
| SEC (US public cos) | 4 business days | SEC Form 8-K |

---

## RECOVER (RC)

Restore capabilities impacted by a cybersecurity incident.

| Category | Key Requirements |
|----------|-----------------|
| RC.RP — Incident Recovery Plan | BCP/DRP documented and tested |
| RC.CO — Incident Recovery Communication | Update stakeholders on progress |

**Recovery Objectives:**
```
RTO (Recovery Time Objective)  → Maximum acceptable downtime
RPO (Recovery Point Objective) → Maximum acceptable data loss
MTD (Maximum Tolerable Downtime) → Point of business failure

Tier 1 (Crown Jewels): RTO < 4h,  RPO < 1h
Tier 2 (Critical):     RTO < 24h, RPO < 4h
Tier 3 (Important):    RTO < 72h, RPO < 24h
Tier 4 (Normal):       RTO < 7d,  RPO < 48h
```

**Backup Testing Requirements:**
```markdown
☐ Backups verified monthly (integrity check)
☐ Full restoration test quarterly (tabletop at minimum)
☐ Offsite / immutable backups (3-2-1 rule: 3 copies, 2 media, 1 offsite)
☐ Backup retention meets regulatory requirements
```

---

## Related Notes

- [[GRC - ISO 27001]] ← Controls framework
- [[GRC - CIS Controls]] ← Technical controls
- [[Risk - Risk Assessment]] ← Detailed risk methodology
- [[White - Instruments]] ← Assessment tools

↩ [[White - Monographs]] · [[White]] · [[Mind Palace]]
