---
type: section
room: yellow
tags: [yellow/notes]
---

# Yellow: Monographs

> [!quote]
> *"Data! Data! Data! I can't make bricks without clay."*
> — *The Adventure of the Copper Beeches*

Secure development, code review, and application security testing. This is the laboratory where Holmes synthesises his compounds — where code is tested before it reaches the world.

---

## Subject Hubs

### OWASP Top 10 (Web)
| Vulnerability | Note | MITRE CWE |
|--------------|------|-----------|
| A01: Broken Access Control | [[OWASP - A01 Broken Access Control]] | CWE-284 |
| A02: Cryptographic Failures | [[OWASP - A02 Cryptographic Failures]] | CWE-327 |
| A03: Injection (SQLi, XSS, etc.) | [[OWASP - A03 Injection]] | CWE-89, CWE-79 |
| A04: Insecure Design | [[OWASP - A04 Insecure Design]] | CWE-657 |
| A05: Security Misconfiguration | [[OWASP - A05 Misconfiguration]] | CWE-16 |
| A06: Vulnerable Components | [[OWASP - A06 Vulnerable Components]] | CWE-1035 |
| A07: Auth & Session Failures | [[OWASP - A07 Auth Failures]] | CWE-287 |
| A08: Software & Data Integrity | [[OWASP - A08 Integrity Failures]] | CWE-494 |
| A09: Security Logging Failures | [[OWASP - A09 Logging Failures]] | CWE-778 |
| A10: SSRF | [[OWASP - A10 SSRF]] | CWE-918 |

### Secure Coding Concepts
| Subject | Note |
|---------|------|
| Input Validation | [[SecCode - Input Validation]] |
| Output Encoding | [[SecCode - Output Encoding]] |
| Authentication Patterns | [[SecCode - Authentication]] |
| Cryptography — Correct Usage | [[SecCode - Cryptography]] |
| Error Handling & Logging | [[SecCode - Error Handling]] |
| Dependency Management | [[SecCode - Dependency Management]] |

### Code Review
| Subject | Note |
|---------|------|
| Code Review Methodology | [[Review - Methodology]] |
| Dangerous Functions Reference | [[Review - Dangerous Functions]] |
| Language-Specific: Python | [[Review - Python]] |
| Language-Specific: JavaScript | [[Review - JavaScript]] |
| Language-Specific: PHP | [[Review - PHP]] |
| Language-Specific: Java | [[Review - Java]] |

---

## OWASP Top 10 — Quick Attack Reference

> For detailed exploitation → see Red room [[TA0001 - Initial Access]]

| Vulnerability | Quick Test | Tool |
|--------------|-----------|------|
| SQLi | `' OR 1=1--` in inputs | sqlmap |
| XSS (Reflected) | `<script>alert(1)</script>` | Burp Suite |
| SSRF | `http://127.0.0.1/admin` in URL params | Burp Collaborator |
| XXE | `<!ENTITY ext SYSTEM "file:///etc/passwd">` | Burp Suite |
| IDOR | Change IDs in requests | Burp Intruder |
| JWT | Alg:none / HS256 with RS256 key | jwt_tool |
| Path Traversal | `../../etc/passwd` | ffuf, manual |
| Command Injection | `; id` `| id` `` `id` `` | manual |

---

## Conventions

- **OWASP notes:** `OWASP - A0X <Name>.md`
- **Secure coding:** `SecCode - <Topic>.md`
- **Code review:** `Review - <Topic>.md`
- **Tags:** `#yellow/<category>` e.g. `#yellow/appsec`, `#yellow/review`

↩ [[Yellow]] · [[Mind Palace]]