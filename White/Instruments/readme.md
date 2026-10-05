---
type: section
room: white
tags: [white/instruments]
---

# White: Instruments

> [!quote]
> *"Mycroft's conclusions were as infallible as so many algebraic propositions."*

Templates, checklists, and reference tools for GRC practitioners. Audit tools with their most-used commands.

---

## 🐧 lynis — Linux Security Audit

**Purpose:** CIS benchmark-aligned hardening audit for Linux/Unix systems.

```bash
# Install
apt install lynis     # Debian/Ubuntu
git clone https://github.com/CISOfy/lynis /opt/lynis

# Full system audit
lynis audit system

# Save report
lynis audit system --report-file /tmp/lynis-report.txt

# Only specific tests (hardening section)
lynis audit system --tests-from-group authentication
lynis audit system --tests-from-group networking
lynis audit system --tests-from-group file-permissions
lynis audit system --tests-from-group malware

# Remote audit (via SSH)
lynis audit system --remote <user>@<host>

# Show hardening index score
grep "hardening_index" /var/log/lynis-report.dat
```

---

## 🔍 OpenSCAP — Compliance Scanning

**Purpose:** NIST SCAP-based compliance scanning (CIS, STIG, PCI-DSS).

```bash
# Install
apt install openscap-scanner scap-security-guide

# List available profiles
oscap info /usr/share/xml/scap/ssg/content/ssg-ubuntu2204-ds.xml | grep "Id: xccdf"

# Scan against CIS profile
oscap xccdf eval \
  --profile xccdf_org.ssgproject.content_profile_cis_level1_server \
  --results /tmp/scan-results.xml \
  --report /tmp/scan-report.html \
  /usr/share/xml/scap/ssg/content/ssg-ubuntu2204-ds.xml

# Scan against STIG
oscap xccdf eval --profile stig ...

# View HTML report
firefox /tmp/scan-report.html
```

---

## 🛡️ CVSS v3.1 Calculator

**Purpose:** Score vulnerabilities for risk prioritisation.

```
# CVSS v3.1 Vector String format:
CVSS:3.1/AV:<av>/AC:<ac>/PR:<pr>/UI:<ui>/S:<s>/C:<c>/I:<i>/A:<a>

# Metric Values:
AV  Attack Vector:      N(etwork) A(djacent) L(ocal) P(hysical)
AC  Attack Complexity:  L(ow) H(igh)
PR  Privileges Req:     N(one) L(ow) H(igh)
UI  User Interaction:   N(one) R(equired)
S   Scope:              U(nchanged) C(hanged)
C   Confidentiality:    N(one) L(ow) H(igh)
I   Integrity:          N(one) L(ow) H(igh)
A   Availability:       N(one) L(ow) H(igh)

# Example — remote unauthenticated RCE, complete impact:
CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H  → Score: 9.8 (Critical)

# Common score examples:
RCE (network, no auth):              ~9.8  Critical
SQLi (authenticated, low priv):      ~6.5  Medium
Stored XSS (auth required):          ~5.4  Medium
Local file read (root required):     ~3.3  Low
```

**Calculator:** [nvd.nist.gov/vuln-metrics/cvss/v3-calculator](https://nvd.nist.gov/vuln-metrics/cvss/v3-calculator)

---

## 📋 Risk Register Template

| Risk ID | Description | Category | Likelihood (1-5) | Impact (1-5) | Score | Owner | Treatment | Status |
|---------|-------------|---------|-----------------|-------------|-------|-------|-----------|--------|
| R-001 | Unpatched critical CVE in web app | Technical | 4 | 5 | 20 | IT | Patch within 7 days | Open |
| R-002 | No MFA on admin accounts | IAM | 3 | 5 | 15 | IT/IAM | Enable MFA | Open |
| R-003 | Sensitive data in unencrypted S3 | Data | 2 | 5 | 10 | Cloud | Enable encryption | Open |

**Risk Score = Likelihood × Impact**
- 20–25: Critical (treat immediately)
- 10–19: High (treat within 30 days)
- 5–9: Medium (treat within 90 days)
- 1–4: Low (accept or treat in backlog)

---

## 📄 Pentest Report Template Structure

```markdown
# Penetration Test Report

## Executive Summary
- Engagement overview, objectives, scope
- Overall risk rating
- Top 3 critical findings (non-technical summary)
- Remediation priority

## Scope and Methodology
- In-scope assets (IPs, domains, applications)
- Out-of-scope
- Testing methodology (PTES, OWASP, etc.)
- Testing dates and testers

## Findings Summary
| ID | Title | Severity | CVSS | Status |
|----|-------|----------|------|--------|
| F-001 | SQL Injection in login | Critical | 9.8 | Open |

## Detailed Findings

### F-001 — SQL Injection in Login Form
**Severity:** Critical | **CVSS:** 9.8
**Affected System:** https://target.com/login
**CWE:** CWE-89

**Description:** ...
**Evidence:** [screenshot/output]
**Impact:** ...
**Remediation:** Use parameterised queries / prepared statements.
**References:** OWASP A03:2021

## Remediation Summary
## Appendix — Tools Used
## Appendix — Methodology Details
```

---

## 🔐 NIST CSF 2.0 Quick Checklist

```markdown
### GOVERN
- [ ] Cybersecurity policy exists and is reviewed annually
- [ ] Roles and responsibilities defined
- [ ] Risk appetite statement documented
- [ ] Supply chain risk management process

### IDENTIFY
- [ ] Asset inventory (hardware, software, data)
- [ ] Risk assessment performed annually
- [ ] Threat modelling for critical systems

### PROTECT
- [ ] MFA enabled for all privileged accounts
- [ ] Least privilege enforced
- [ ] Patch management process (critical: <7 days, high: <30 days)
- [ ] Security awareness training
- [ ] Data classified and encrypted at rest and in transit
- [ ] Endpoint protection deployed

### DETECT
- [ ] SIEM deployed and tuned
- [ ] Log retention ≥ 90 days (12 months for regulated)
- [ ] EDR deployed on all endpoints
- [ ] Vulnerability scanning monthly or continuous
- [ ] Alerting on critical IOCs

### RESPOND
- [ ] Incident response plan documented and tested
- [ ] IR retainer or team in place
- [ ] Communication plan (legal, PR, regulatory)

### RECOVER
- [ ] Business continuity plan tested
- [ ] Backups tested for restoration quarterly
- [ ] RTO and RPO defined and achievable
```

---

## Conventions

- Templates named: `Template - <Name>.md`
- Tags: `#white/tool`, `#white/template`, `#white/compliance`

↩ [[White]] · [[Mind Palace]]