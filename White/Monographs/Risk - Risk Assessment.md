---
type: monograph
room: white
subject: "Risk Assessment Methodology"
tags: [white/risk, white/grc]
date_updated: {{date}}
---

# Risk — Risk Assessment Methodology

> [!quote]
> *"The world is full of obvious things which nobody by any chance ever observes."*
> — *The Hound of the Baskervilles*

**Framework:** ISO 27005 / NIST SP 800-30 | **Phase:** Identify (NIST CSF)

Risk assessment is the systematic process of understanding what could go wrong, how likely it is, what the impact would be, and what to do about it. Without it, security spending is guesswork.

---

## Risk Equation

```
Risk = Threat × Vulnerability × Impact

OR simplified:

Risk Score = Likelihood (1–5) × Impact (1–5) = 1–25
```

---

## Step-by-Step Process

### Step 1 — Define Scope
```markdown
- What systems / processes / data are in scope?
- What is the business context? (revenue impact, regulatory exposure)
- What are the crown jewels? (most critical assets)
- Time horizon: next 12 months
```

### Step 2 — Asset Inventory
```markdown
| Asset ID | Asset Name | Owner | Classification | Location | Business Value |
|----------|-----------|-------|----------------|---------|----------------|
| A-001 | Customer DB | CTO | Restricted | AWS RDS | Critical |
| A-002 | HR System | CHRO | Confidential | On-prem | High |
| A-003 | Public Website | Marketing | Public | Cloudflare | Medium |
```

**Data Classification:**
| Level | Description | Examples |
|-------|-------------|---------|
| Restricted | Highest sensitivity — breach = regulatory/legal consequence | PII, PCI, PHI, trade secrets |
| Confidential | Internal use only — breach = reputational/financial harm | Financial data, IP, M&A |
| Internal | General internal use | Policies, internal docs |
| Public | Intentionally public | Marketing, press releases |

### Step 3 — Threat Identification
Use threat catalogues to systematically identify what could go wrong:

```
Threat Sources:
  - External adversary (nation-state, criminal, hacktivist)
  - Insider (malicious, negligent, accidental)
  - Natural disaster / physical (flood, fire, power)
  - Supply chain / third party

Threat Events (using STRIDE):
  Spoofing        → Identity forgery (T1078 Valid Accounts)
  Tampering       → Data modification (T1485 Data Destruction)
  Repudiation     → Deny actions (T1070 Indicator Removal)
  Info Disclosure → Data breach (T1041 Exfil over C2)
  DoS             → Availability (T1499 Endpoint DoS)
  Elevation       → Privilege escalation (T1068)
```

### Step 4 — Vulnerability Identification
```markdown
Sources:
☐ Vulnerability scanner results (Nessus, Qualys, OpenVAS)
☐ Penetration test findings
☐ Code review / SAST results
☐ Configuration review (CIS Benchmark score)
☐ Security awareness survey results
☐ Third-party audit findings
☐ CVE feeds for installed software
☐ Industry threat intelligence (ISACs, CERT advisories)
```

### Step 5 — Likelihood Rating

| Score | Likelihood | Description |
|-------|-----------|-------------|
| 5 | Almost Certain | Likely to occur multiple times per year |
| 4 | Likely | Has occurred in similar organisations; expected within a year |
| 3 | Possible | Has occurred in the industry; possible within 3 years |
| 2 | Unlikely | Would require specific circumstances; unlikely within 5 years |
| 1 | Rare | Only in exceptional circumstances |

**Factors increasing likelihood:**
- Publicly known CVE for an unpatched system
- No MFA on internet-facing services
- Known threat actor targeting your sector
- Recent publicly disclosed similar breach

### Step 6 — Impact Rating

| Score | Impact | Description |
|-------|--------|-------------|
| 5 | Catastrophic | Business failure, massive regulatory fine, criminal liability |
| 4 | Major | Significant financial loss, regulatory sanction, major reputational damage |
| 3 | Moderate | Some financial loss, minor regulatory attention, operational disruption |
| 2 | Minor | Limited financial impact, recoverable within days |
| 1 | Insignificant | Negligible impact, recoverable immediately |

**Impact dimensions:** Financial · Legal/Regulatory · Reputational · Operational · Safety

### Step 7 — Risk Scoring Matrix

```
         │  Impact 1  │  Impact 2  │  Impact 3  │  Impact 4  │  Impact 5
─────────┼────────────┼────────────┼────────────┼────────────┼────────────
Likely 5 │     5      │    10      │    15      │    20      │    25
Likely 4 │     4      │     8      │    12      │    16      │    20
Likely 3 │     3      │     6      │     9      │    12      │    15
Likely 2 │     2      │     4      │     6      │     8      │    10
Likely 1 │     1      │     2      │     3      │     4      │     5
```

| Score | Rating | Treatment SLA |
|-------|--------|---------------|
| 20–25 | 🔴 Critical | Treat immediately (within 7 days) |
| 10–19 | 🟠 High | Treat within 30 days |
| 5–9 | 🟡 Medium | Treat within 90 days |
| 1–4 | 🟢 Low | Accept or treat in next cycle |

### Step 8 — Risk Treatment Options

| Option | When to Use | Example |
|--------|------------|---------|
| **Mitigate** | Control reduces risk to acceptable level | Patch, add MFA, segment network |
| **Transfer** | Residual risk is shared with a third party | Cyber insurance, outsource DR |
| **Avoid** | Stop the activity causing the risk | Don't collect unnecessary PII |
| **Accept** | Risk is below appetite or cost > benefit | Accept low-likelihood minor risk |

### Step 9 — Risk Register

```markdown
| Risk ID | Title | Asset | Threat | Vuln | L | I | Score | Rating | Treatment | Owner | Due | Residual | Status |
|---------|-------|-------|--------|------|---|---|-------|--------|-----------|-------|-----|----------|--------|
| R-001 | Unpatched CVE-2024-XXXX | Web App | External attacker | Missing patch | 4 | 5 | 20 | Critical | Patch immediately | IT | 7 days | TBD | Open |
| R-002 | No MFA on VPN | Remote access | Credential theft | No MFA | 4 | 4 | 16 | High | Enable MFA | IT/IAM | 30 days | Low | In Progress |
```

### Step 10 — Review Cycle
```markdown
☐ Critical risks: review weekly until treated
☐ High risks: monthly review
☐ Full risk assessment: annually or after significant change
☐ Trigger events: major breach, significant architecture change, acquisition
```

---

## Threat Modelling (STRIDE per Component)

For application-level risk, use STRIDE against each component:

```
Component: Login Endpoint
─────────────────────────
S — Spoofing:        Can an attacker impersonate a valid user? (brute force, stolen creds)
T — Tampering:       Can the request/response be modified? (HTTP downgrade, MITM)
R — Repudiation:     Are user actions logged? (audit trail gaps)
I — Info Disclosure: Does the response reveal sensitive data? (stack trace, verbose errors)
D — DoS:             Can the endpoint be overwhelmed? (no rate limiting)
E — Elevation:       Can a low-priv user gain admin access? (IDOR, broken auth)
```

---

## Related Notes

- [[GRC - NIST CSF 2.0]] ← Framework context
- [[GRC - ISO 27001]] ← Control selection after risk
- [[Audit - CVSS]] ← Technical vulnerability scoring
- [[White - Instruments]] ← Tools to support risk assessment

↩ [[White - Monographs]] · [[White]] · [[Mind Palace]]
