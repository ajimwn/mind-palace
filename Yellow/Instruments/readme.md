---
type: section
room: yellow
tags: [yellow/instruments]
---

# Yellow: Instruments

> [!quote]
> *"Holmes was working hard over a chemical investigation."*

Static analysis, DAST, dependency scanning, secrets detection — with the actual commands you reach for during a code review or secure build audit.

---

## 🔬 semgrep — Static Analysis (Multi-language)

**Purpose:** Pattern-based SAST — finds security bugs using community and custom rules.

```bash
# Install
pip install semgrep

# Scan with OWASP Top 10 rules
semgrep --config=p/owasp-top-ten ./src

# Scan with all security rules
semgrep --config=p/r2c-security-audit ./src

# Scan for specific language
semgrep --config=p/python ./src
semgrep --config=p/javascript ./src
semgrep --config=p/java ./src
semgrep --config=p/php ./src

# Run in CI mode (exit 1 if findings)
semgrep ci --config=p/default

# Write custom rule and test
cat > rule.yaml << 'EOF'
rules:
  - id: hardcoded-password
    pattern: password = "..."
    message: Hardcoded password found
    languages: [python]
    severity: ERROR
EOF
semgrep --config rule.yaml ./src

# Output as JSON
semgrep --config=p/owasp-top-ten ./src --json > results.json

# Suppress a finding (add to code)
# nosemgrep: rule-id
```

---

## 🐍 bandit — Python Security Scanner

**Purpose:** Find common Python security issues (SAST).

```bash
# Scan a file
bandit suspicious.py

# Scan a directory
bandit -r ./src

# Only show high severity, high confidence
bandit -r ./src -ll -ii

# Exclude test directories
bandit -r ./src --exclude ./tests,./venv

# Output formats
bandit -r ./src -f txt -o report.txt     # Text
bandit -r ./src -f json -o report.json   # JSON
bandit -r ./src -f html -o report.html   # HTML

# Test by category
bandit -r ./src -t B101  # assert_used
bandit -r ./src -t B602  # subprocess_popen_with_shell_equals_true
bandit -r ./src -t B501  # request_with_no_cert_validation

# Common test IDs
# B101 assert_used          B201 flask_debug_true
# B301 pickle               B302 marshal
# B307 eval                 B501 request_with_no_cert_validation
# B601 paramiko_calls       B602 subprocess_popen_with_shell
# B608 hardcoded_sql_expressions
```

---

## 🔑 gitleaks — Secrets Detection

**Purpose:** Find secrets, API keys, passwords committed to git repos.

```bash
# Scan current repo (full history)
gitleaks detect --source . --verbose

# Scan only staged changes (pre-commit hook)
gitleaks protect --staged --verbose

# Scan specific branch
gitleaks detect --source . --log-opts="main..feature-branch"

# Generate a baseline (ignore existing findings)
gitleaks detect --source . --baseline-path baseline.json
gitleaks detect --source . --baseline-path baseline.json  # will only show new

# Output as JSON
gitleaks detect --source . --report-format json --report-path leaks.json

# Suppress false positive (add to .gitleaks.toml)
# [extend]
# [[rules]]
# id = "rule-to-ignore"
# [allowlist]
# paths = ["test/fixtures/"]
```

---

## 🔍 trufflehog — Deep Git History Scan

**Purpose:** Find verified secrets in git history — more thorough than gitleaks.

```bash
# Scan local repo
trufflehog git file://. --only-verified

# Scan with more workers
trufflehog git file://. --only-verified --concurrency=10

# Scan a GitHub repo
trufflehog github --repo https://github.com/org/repo --only-verified

# Scan specific branch
trufflehog git file://. --branch main --only-verified

# JSON output
trufflehog git file://. --only-verified --json

# Scan Docker image
trufflehog docker --image nginx:latest
```

---

## 🐋 trivy — Container + IaC Scanner

**Purpose:** All-in-one scanner for images, filesystems, repos, and IaC.

```bash
# Scan Docker image (default: vulns + secrets)
trivy image nginx:latest

# Scan with severity filter
trivy image nginx:latest --severity HIGH,CRITICAL

# Scan and output JSON
trivy image nginx:latest -f json -o trivy-results.json

# Scan local filesystem
trivy fs ./src --scanners vuln,secret,config

# Scan IaC (Terraform, K8s, Dockerfile)
trivy config ./terraform/
trivy config ./k8s/manifests/
trivy config ./Dockerfile

# Scan a git repo
trivy repo https://github.com/org/repo

# Generate SBOM (Software Bill of Materials)
trivy image --format cyclonedx --output sbom.json nginx:latest

# Scan with custom ignore file (.trivyignore)
echo "CVE-2021-1234" >> .trivyignore
trivy image nginx:latest
```

---

## 🕷️ OWASP ZAP — Web DAST

**Purpose:** Automated web application vulnerability scanning.

```bash
# Docker — quick baseline scan
docker run -t owasp/zap2docker-stable zap-baseline.py \
  -t https://target.com \
  -r zap-report.html

# Full active scan (more thorough, more aggressive)
docker run -t owasp/zap2docker-stable zap-full-scan.py \
  -t https://target.com \
  -r zap-full-report.html

# API scan (OpenAPI/Swagger)
docker run -t owasp/zap2docker-stable zap-api-scan.py \
  -t https://target.com/openapi.json \
  -f openapi \
  -r zap-api-report.html

# ZAP CLI (from within Docker)
docker run -i owasp/zap2docker-stable zap-cli \
  quick-scan --self-contained --start-options '-config api.disablekey=true' \
  https://target.com
```

---

## 🧩 nikto — Web Server Scanner

**Purpose:** Detect misconfigurations, outdated software, dangerous files.

```bash
# Basic scan
nikto -h http://target.com

# Scan with SSL
nikto -h https://target.com -ssl

# Scan specific port
nikto -h target.com -port 8080

# Save output
nikto -h http://target.com -o nikto-report.txt -Format txt
nikto -h http://target.com -o nikto-report.html -Format html

# Tune (what to test)
nikto -h http://target.com -Tuning 9  # 9 = only SQL injection tests
# Tuning values: 0=file upload 1=interesting file 2=misconfiguration 4=injection 6=traversal 7=RCE

# With cookie
nikto -h http://target.com -Cgidirs all -id "admin:password" -c "session=abc123"
```

---

## 🔐 jwt_tool — JWT Testing

**Purpose:** Analyse, decode, and attack JSON Web Tokens.

```bash
# Install
pip install jwt_tool

# Decode and analyse a JWT
python3 jwt_tool.py <jwt_token>

# Test all attacks automatically
python3 jwt_tool.py <jwt> -M at -t "https://target.com/api/protected"

# Algorithm confusion (RS256 → HS256)
python3 jwt_tool.py <jwt> -X k -pk server.pem

# None algorithm attack
python3 jwt_tool.py <jwt> -X a

# Brute-force secret
python3 jwt_tool.py <jwt> -C -d /usr/share/wordlists/rockyou.txt

# Tamper payload field
python3 jwt_tool.py <jwt> -T -S hs256 -p "secret"
```

---

## Dangerous Functions Quick-Reference

```python
# Python — AVOID these
os.system(user_input)             # → use subprocess with list args
subprocess.call(cmd, shell=True)  # → shell=False
eval(user_input)                  # → never eval untrusted input
pickle.loads(data)                # → use json.loads
yaml.load(data)                   # → yaml.safe_load
```
```php
// PHP — AVOID
eval($input); system($input); exec($input); passthru($input);
include($input); require($input);  // → validate strictly
```
```javascript
// JS / Node.js — AVOID
eval(input); new Function(input)();
child_process.exec(input);         // → use execFile with args array
innerHTML = input;                 // → use textContent or sanitise
```

↩ [[Yellow]] · [[Mind Palace]]