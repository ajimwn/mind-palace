---
type: section
room: green
tags: [green/monographs]
---

# Green: Monographs

> [!quote]
> *"It is a capital mistake to theorize before one has data."*
> — Cloud misconfigs are data. Audit everything.

Cloud security and DevSecOps studies. Green covers the intersection of infrastructure, containers, and pipelines — attack surfaces that didn't exist a decade ago.

---

## Study Areas

```
AWS SECURITY              CONTAINER SECURITY      CI/CD SECURITY
IAM misconfigs            Docker escape           PPE attacks
IMDS SSRF                 K8s privilege esc.      Secrets in CI
S3 public access          Image supply chain      SLSA provenance
CloudTrail auditing       RBAC hardening          Dependency confusion
        ↓                       ↓                       ↓
AZURE / GCP               INFRASTRUCTURE AS CODE  THREAT DETECTION
Similar IAM patterns      Terraform misconfig     GuardDuty, Defender
Managed identity          checkov scanning        CloudTrail + SIEM
```

---

## Monographs

| Subject | Focus | Status |
|---------|-------|--------|
| [[Cloud - AWS Security]] | IAM, IMDS, S3, CloudTrail, attack paths | ✅ Ready |
| [[Cloud - Container Security]] | Docker escape, K8s misconfigs, image security | ✅ Ready |
| [[Cloud - CI CD Security]] | PPE, secrets, SLSA, GitHub Actions hardening | ✅ Ready |

---

## Attack Path Quick Reference

```
Public S3 → credentials → IAM privesc → full account
SSRF on EC2 → IMDSv1 → role creds → AWS API
Compromised CI → OIDC token → AWS/GCP deploy perms → prod
Privileged pod → nsenter → host OS → all other pods
Docker socket mount → docker run privileged → host escape
Fork PR → poison workflow → exfil repo secrets
```

---

## Conventions

- **Cloud provider guides:** `Cloud - <Provider/Topic>.md`
- **Tool commands:** `→ [[Green - Instruments]]`
- **Tags:** `#green/<topic>` e.g. `#green/aws`, `#green/containers`, `#green/cicd`

↩ [[Green]] · [[Mind Palace]]