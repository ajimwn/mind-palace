---
type: section
room: yellow
tags: [yellow/monographs]
---

# Yellow: Monographs

> [!quote]
> *"The world is full of obvious things which nobody by any chance ever observes."*
> — Every bug is obvious in hindsight.

Application security studies spanning OWASP, secure code review, authentication attacks, and DevSecOps tooling. Yellow is the developer's room — security through the lens of code.

---

## Study Areas

```
SECURE CODE REVIEW        OWASP TOP 10          AUTH & TOKENS
Source → Sink analysis    A01 Access Control    JWT attacks
SAST tooling              A03 Injection         OAuth 2.0 flaws
Dangerous patterns        A07 Auth Failures     Session security
        ↓                       ↓                      ↓
DEPENDENCY SECURITY       API SECURITY           PIPELINE SECURITY
SCA / supply chain        GraphQL, REST          CI/CD (→ Green room)
Typosquatting             Rate limiting          SBOM / SLSA
```

---

## Monographs

| Subject | Focus | Status |
|---------|-------|--------|
| [[AppSec - OWASP Top 10]] | Full A01–A10 — attack + defence | ✅ Ready |
| [[AppSec - Secure Code Review]] | Source-to-sink methodology, SAST | ✅ Ready |
| [[AppSec - JWT & Auth]] | JWT attacks, jwt_tool, OAuth flaws | ✅ Ready |

---

## OWASP Top 10 Quick Index

| ID | Name | Key Attack |
|----|------|-----------|
| A01 | Broken Access Control | IDOR, forced browsing |
| A02 | Cryptographic Failures | Weak TLS, exposed secrets |
| A03 | Injection | SQLi, XSS, CMDi |
| A04 | Insecure Design | Missing rate limits, no threat model |
| A05 | Security Misconfiguration | Default creds, verbose errors |
| A06 | Vulnerable Components | Outdated deps with known CVEs |
| A07 | Auth & Session Failures | Brute force, weak sessions, JWT |
| A08 | Software & Data Integrity | Deserialization, unsigned artifacts |
| A09 | Logging & Monitoring Failures | No audit trail, no alerts |
| A10 | SSRF | Cloud metadata exfil |

---

## Conventions

- **OWASP monographs:** `AppSec - <Topic>.md`
- **Tool guides:** `→ [[Yellow - Instruments]]`
- **Tags:** `#yellow/<topic>` e.g. `#yellow/owasp`, `#yellow/appsec`, `#yellow/auth`

↩ [[Yellow]] · [[Mind Palace]]