---
type: section
room: yellow
tags: [yellow/tools]
---

# Yellow: Instruments

> [!quote]
> *"Holmes was working hard over a chemical investigation."*

Static analysis, DAST, dependency scanning, and secrets detection tools for the laboratory.

---

## Inventory by Stage

### Static Analysis (SAST)
| Tool | Language | Note |
|------|---------|------|
| `semgrep` | Multi-language | [[semgrep]] |
| `bandit` | Python | [[bandit]] |
| `eslint-security` | JavaScript/TS | [[eslint-security]] |
| `gosec` | Go | [[gosec]] |
| `findbugs` / SpotBugs | Java | [[spotbugs]] |
| `sonarqube` | Multi-language | [[sonarqube]] |
| `flawfinder` | C/C++ | [[flawfinder]] |

### Dynamic Analysis (DAST)
| Tool | Role | Note |
|------|------|------|
| `burpsuite` | Web application testing | [[burpsuite]] |
| `zaproxy` (OWASP ZAP) | Automated web scan | [[zaproxy]] |
| `nikto` | Web server scanner | [[nikto]] |
| `wfuzz` | Parameter fuzzing | [[wfuzz]] |
| `ffuf` | Content discovery | [[ffuf]] |
| `sqlmap` | SQLi automation | [[sqlmap]] |
| `jwt_tool` | JWT attack tool | [[jwt-tool]] |

### Secrets Detection
| Tool | Role | Note |
|------|------|------|
| `truffleHog` | Git history secrets | [[trufflehog]] |
| `gitleaks` | Secrets in git repos | [[gitleaks]] |
| `detect-secrets` | Pre-commit hook | [[detect-secrets]] |

### Dependency Scanning (SCA)
| Tool | Language | Note |
|------|---------|------|
| `safety` | Python | [[safety]] |
| `npm audit` | JavaScript | Built-in |
| `dependency-check` | Multi-language | [[owasp-dc]] |
| `snyk` | Multi-language | [[snyk]] |
| `grype` | Container / OS | [[grype]] |

### Container & IaC Security
| Tool | Role | Note |
|------|------|------|
| `trivy` | Container + IaC scanning | [[trivy]] |
| `checkov` | IaC policy-as-code | [[checkov]] |
| `tfsec` | Terraform security | [[tfsec]] |
| `hadolint` | Dockerfile linting | [[hadolint]] |
| `kube-bench` | Kubernetes CIS benchmark | [[kube-bench]] |

---

## Quick Commands

```bash
# Semgrep — scan with OWASP ruleset
semgrep --config=p/owasp-top-ten ./src

# Bandit — Python security scan
bandit -r ./src -ll -f txt

# TruffleHog — scan git history for secrets
trufflehog git file://. --only-verified

# Gitleaks — pre-commit / CI scan
gitleaks detect --source . --verbose

# Trivy — scan container image
trivy image <image>:<tag>

# Trivy — scan filesystem / IaC
trivy fs . --scanners vuln,secret,config

# Nikto — web server scan
nikto -h <url> -o nikto-report.txt

# OWASP ZAP — quick active scan
docker run -t owasp/zap2docker-stable zap-baseline.py -t <url>
```

---

## Dangerous Function Reference (Quick)

### Python
```python
# AVOID — command injection risk
os.system(user_input)
subprocess.call(user_input, shell=True)
eval(user_input)
exec(user_input)
pickle.loads(user_data)          # arbitrary code execution
yaml.load(data)                  # use yaml.safe_load()
```

### PHP
```php
// AVOID — code/command injection
eval($user_input);
system($user_input);
exec($user_input);
passthru($user_input);
shell_exec($user_input);
include($user_input);            // LFI/RFI
file_get_contents($user_input);  // SSRF
```

### JavaScript (Node.js)
```javascript
// AVOID
eval(userInput);
new Function(userInput)();
child_process.exec(userInput);    // command injection
res.send(userInput);              // XSS if not encoded
```

---

## Conventions

- **One note per tool** named after the tool
- **Template:** [[Templates/Yellow - SAST Finding]] for code review findings
- **Tags:** `#yellow/tool`, `#yellow/sast`, `#yellow/dast`

↩ [[Yellow]] · [[Mind Palace]]