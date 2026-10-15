---
type: monograph
room: yellow
subject: "Secure Code Review"
tags: [yellow/appsec, yellow/code-review]
date_updated: {{date}}
---

# AppSec — Secure Code Review

> [!quote]
> *"When you have eliminated the impossible, whatever remains, however improbable, must be the truth."*
> — Start with the most dangerous sinks; work backwards to the sources.

**Phase:** Shift-Left | **Standards:** OWASP Code Review Guide, CWE Top 25

Secure code review is the process of finding vulnerabilities in source code before they reach production. It combines manual reasoning with automated tooling — neither alone is sufficient.

---

## Methodology: Source → Sink

```
1. IDENTIFY ENTRY POINTS (Sources)
   → HTTP request parameters, headers, cookies, URL path
   → File uploads, WebSocket messages, GraphQL variables
   → Environment variables, config files, database values
   → Inter-process communication (IPC), message queues

2. TRACE DATA FLOW
   → Follow user input through the application
   → Note all transformations (encoding, type casting, sanitisation)
   → Identify where sanitisation is absent or bypassable

3. REACH DANGEROUS SINKS
   → SQL queries, OS commands, eval(), HTML output, file paths
   → Deserialisation, LDAP queries, XPath, template engines

4. VERIFY CONTROLS
   → Is input validated? (allowlist > blocklist)
   → Is output encoded? (context-aware)
   → Is parameterisation used? (not string concat)
```

---

## High-Value Code Patterns to Search

### SQL Injection Sinks
```bash
# Search for string concatenation into SQL (Python)
grep -rn "execute\s*(" --include="*.py" . | grep -v "?"   # missing ? placeholder
grep -rn "f\"SELECT\|f'SELECT\|%.*SELECT\|+.*WHERE" --include="*.py" .

# PHP
grep -rn "mysql_query\|mysqli_query\|PDO::query" --include="*.php" . | grep -v "prepare"

# Java
grep -rn "createQuery\|executeQuery\|prepareStatement" --include="*.java" . | grep "\+"

# Node.js
grep -rn "db\.query\|connection\.query" --include="*.js" . | grep "\`\|+"
```

### Command Injection Sinks
```bash
# Python
grep -rn "os\.system\|subprocess\.call\|subprocess\.Popen\|eval\|exec" --include="*.py" .

# PHP
grep -rn "system(\|exec(\|shell_exec(\|passthru(\|popen(\|eval(" --include="*.php" .

# Node.js
grep -rn "child_process\|exec(\|spawn(" --include="*.js" .

# Ruby
grep -rn "system(\|exec(\|`\|IO\.popen\|eval" --include="*.rb" .
```

### XSS Sinks
```bash
# JavaScript / React
grep -rn "dangerouslySetInnerHTML\|innerHTML\|document\.write\|eval(" --include="*.js" --include="*.jsx" --include="*.tsx" .

# Python (Flask/Django templates)
grep -rn "Markup(\|mark_safe(\|format_html(" --include="*.py" --include="*.html" .

# PHP
grep -rn "echo \$\|print \$\|<?=" --include="*.php" . | grep -v "htmlspecialchars\|htmlentities"
```

### Insecure Deserialisation
```bash
# Python
grep -rn "pickle\.loads\|yaml\.load\|marshal\.loads" --include="*.py" .
# Fix: yaml.safe_load() instead of yaml.load()

# PHP
grep -rn "unserialize(" --include="*.php" .

# Java
grep -rn "ObjectInputStream\|readObject(" --include="*.java" .

# Node.js
grep -rn "JSON\.parse\|eval(" --include="*.js" .  # eval of JSON is dangerous
```

### Hardcoded Secrets
```bash
# Scan for secrets in code
gitleaks detect --source . --verbose
trufflehog filesystem .

# Grep patterns
grep -rn "password\s*=\s*['\"]" --include="*.py" --include="*.php" --include="*.js" .
grep -rn "api_key\s*=\|apikey\s*=\|secret\s*=\|token\s*=" --include="*.py" . | grep -v "os\.environ\|getenv\|config"
grep -rn "BEGIN RSA PRIVATE KEY\|AKIA[0-9A-Z]\{16\}" -r .

# AWS keys
grep -rn "AKIA" -r . --include="*.py" --include="*.js" --include="*.env" --include="*.json"
```

---

## Automated Static Analysis

```bash
# Semgrep — rule-based SAST
semgrep --config=p/owasp-top-ten .
semgrep --config=p/python .
semgrep --config=p/javascript .
semgrep --config=p/java .
semgrep --config=auto .           # auto-select rules for detected languages
semgrep --config=p/secrets .      # secret detection

# Bandit — Python security linter
bandit -r . -f txt -o bandit-report.txt
bandit -r . -ll   # only medium and high severity
bandit -r . -t B602,B603,B608  # specific checks: subprocess, SQLi

# Safety — Python dependency vulnerabilities
pip install safety
safety check
safety check -r requirements.txt

# npm audit — Node.js
npm audit
npm audit --audit-level=high

# OWASP Dependency Check
dependency-check --project "MyApp" --scan ./ --format HTML
```

---

## Code Review Checklist

```markdown
### Authentication & Session
☐ Passwords hashed with bcrypt/Argon2 (not MD5/SHA1/SHA256 unsalted)
☐ Session tokens: ≥128 bit random, regenerated after login
☐ Session expiry enforced server-side (idle + absolute)
☐ Secure + HttpOnly + SameSite flags on session cookies

### Authorisation
☐ Every endpoint checks permission server-side — no client-side trust
☐ Object IDs: server validates that caller owns the object (IDOR check)
☐ Sensitive functions restricted by role, not just URL obscurity

### Input Validation
☐ Allowlist validation for all user input
☐ Parameterised queries for all DB access — zero string concatenation
☐ File upload: whitelist MIME + extension, rename, store outside webroot

### Output Encoding
☐ HTML output: htmlspecialchars / html.escape / context-aware encoding
☐ JS context: JSON.stringify or explicit escape — not direct string embed
☐ SQL: parameterised only

### Cryptography
☐ TLS 1.2+ on all connections — no unencrypted fallback
☐ Secrets in environment variables or secret manager — not source code
☐ No custom crypto implementations

### Error Handling
☐ No stack traces exposed to end users
☐ Errors log server-side with correlation ID — return generic message to client
☐ No sensitive data (tokens, passwords, PII) in log files
```

---

## Related Notes

- [[AppSec - OWASP Top 10]] ← Vulnerability category context
- [[AppSec - JWT & Auth]] ← Token and session attack deep-dive
- [[Yellow - Instruments]] ← Semgrep, bandit, gitleaks, ZAP commands

↩ [[Yellow - Monographs]] · [[Yellow]] · [[Mind Palace]]
