---
type: monograph
room: green
subject: "AWS Security"
tags: [green/cloud, green/aws]
date_updated: {{date}}
---

# Cloud — AWS Security

> [!quote]
> *"You know my methods, Watson. There was not one of them which I did not apply to the inquiry."*

**Reference:** AWS Well-Architected Security Pillar | **Phase:** Identify + Protect (NIST CSF)

AWS security is built on the **Shared Responsibility Model**: AWS secures the cloud infrastructure; you secure what's *in* the cloud (data, IAM, network config, applications).

---

## Shared Responsibility Model

```
AWS Responsibility (Security OF the cloud):
  ├── Physical datacentre security
  ├── Hardware, networking, virtualisation layer
  └── Managed services (RDS engine patching, Lambda runtime)

Customer Responsibility (Security IN the cloud):
  ├── IAM — who can do what
  ├── Data encryption (at rest + transit)
  ├── Network config — VPC, security groups, NACLs
  ├── OS and app patching (EC2)
  └── Audit logging (CloudTrail, Config)
```

---

## IAM — The #1 Attack Surface

### Common IAM Misconfigurations
```bash
# Check for overly permissive policies
aws iam list-policies --scope Local
aws iam get-policy-version --policy-arn <arn> --version-id v1

# Check for inline policies (dangerous — bypass SCPs)
aws iam list-user-policies --user-name <user>
aws iam get-user-policy --user-name <user> --policy-name <policy>

# Find users with AdministratorAccess
aws iam list-entities-for-policy \
  --policy-arn arn:aws:iam::aws:policy/AdministratorAccess

# Check for access keys older than 90 days
aws iam generate-credential-report
aws iam get-credential-report --output text --query Content | base64 -d | \
  awk -F',' 'NR>1 && $10!="N/A" {print $1, $10}'  # access_key_1_last_used_date

# Check which services an identity can access (IAM simulator)
aws iam simulate-principal-policy \
  --policy-source-arn arn:aws:iam::123456789:user/testuser \
  --action-names s3:GetObject ec2:DescribeInstances

# Enumerate role trust relationships
aws iam list-roles --query 'Roles[*].{Name:RoleName,Trust:AssumeRolePolicyDocument}'
```

### IAM Best Practices Checklist
```markdown
☐ Root account: no access keys, MFA enabled, not used for daily work
☐ All IAM users: MFA enforced (via SCP or permission boundary)
☐ Principle of least privilege: all policies scoped to minimum actions/resources
☐ No wildcards (*) on Action or Resource in production policies
☐ Access keys rotated every 90 days — old keys deactivated
☐ Service roles use Instance Profiles (not access keys on EC2)
☐ Cross-account access uses role assumption with external ID, not shared keys
☐ Privileged access uses temporary credentials (STS AssumeRole)
```

---

## IMDS — Metadata Service Attacks

```bash
# IMDSv1 — vulnerable to SSRF (no auth required)
# If you can make the server issue this HTTP request:
curl http://169.254.169.254/latest/meta-data/

# Get temporary credentials from IMDS
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/<role-name>
# Returns: AccessKeyId, SecretAccessKey, Token — valid for ~1h

# IMDSv2 — requires PUT token first (protects against SSRF)
TOKEN=$(curl -s -X PUT http://169.254.169.254/latest/api/token \
  -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")
curl -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/iam/security-credentials/

# Use stolen credentials from non-AWS machine
export AWS_ACCESS_KEY_ID=ASIA...
export AWS_SECRET_ACCESS_KEY=...
export AWS_SESSION_TOKEN=...
aws sts get-caller-identity  # confirm identity
```

**Fix:**
```
✅ Enforce IMDSv2 on all EC2 instances:
aws ec2 modify-instance-metadata-options \
  --instance-id i-xxxx \
  --http-tokens required \
  --http-endpoint enabled
```

---

## S3 — Common Misconfigurations

```bash
# Find public buckets (unauthenticated)
aws s3 ls s3://target-bucket-name --no-sign-request
aws s3 cp s3://target-bucket-name/sensitive.pdf . --no-sign-request

# Check bucket ACL and policy
aws s3api get-bucket-acl --bucket <bucket>
aws s3api get-bucket-policy --bucket <bucket>
aws s3api get-public-access-block --bucket <bucket>

# Check for bucket logging
aws s3api get-bucket-logging --bucket <bucket>

# Enumerate buckets (requires credentials)
aws s3 ls

# Subdomain takeover — find buckets via CNAME
# If CNAME points to s3.amazonaws.com but bucket doesn't exist → claim it
```

**S3 Hardening:**
```markdown
☐ Block All Public Access: enabled (S3 account-level setting)
☐ Bucket versioning: enabled (protects against ransomware)
☐ Bucket logging: enabled — logs to separate audit bucket
☐ Server-side encryption: SSE-S3 or SSE-KMS
☐ MFA Delete: enabled for versioned buckets containing sensitive data
☐ S3 Object Lock: WORM for compliance/immutable logs
```

---

## CloudTrail — Audit Logging

```bash
# Check if CloudTrail is enabled in all regions
aws cloudtrail describe-trails --include-shadow-trails
aws cloudtrail get-trail-status --name <trail-name>

# Check for log file validation (integrity)
aws cloudtrail get-trail --name <trail> --query 'Trail.LogFileValidationEnabled'

# Query CloudTrail events
aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=EventName,AttributeValue=ConsoleLogin \
  --start-time 2024-01-01 --end-time 2024-01-02

# Find recent root account usage
aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=Username,AttributeValue=root \
  --max-results 10

# Detect IAM key creation
aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=EventName,AttributeValue=CreateAccessKey
```

**CloudTrail Hardening:**
```markdown
☐ Multi-region trail enabled covering all regions
☐ Management + Data events enabled (data events = S3 object access, Lambda)
☐ CloudWatch Logs integration — enables alerting on CloudTrail events
☐ S3 bucket for CloudTrail: access logging enabled, versioning, MFA delete
☐ Log file validation: enabled (detects tampering)
☐ CloudTrail Insights: enabled (anomaly detection)
```

---

## Network — VPC Security

```bash
# Check security groups for 0.0.0.0/0 ingress
aws ec2 describe-security-groups \
  --query 'SecurityGroups[?IpPermissions[?IpRanges[?CidrIp==`0.0.0.0/0`]]].{ID:GroupId,Name:GroupName}'

# Check for unrestricted SSH (port 22) or RDP (port 3389) from internet
aws ec2 describe-security-groups \
  --filters "Name=ip-permission.from-port,Values=22" \
            "Name=ip-permission.cidr,Values=0.0.0.0/0" \
  --query 'SecurityGroups[*].GroupId'

# Check VPC Flow Logs status
aws ec2 describe-flow-logs

# Check NACLs
aws ec2 describe-network-acls
```

**Network Hardening:**
```markdown
☐ No 0.0.0.0/0 on SSH (22) or RDP (3389) — use bastion host or SSM Session Manager
☐ VPC Flow Logs enabled for all VPCs — stored to S3 or CloudWatch
☐ Private subnets for databases and application servers
☐ NAT Gateway (not IGW) for outbound internet from private subnets
☐ Security Group: least-privilege, specific CIDR ranges
☐ AWS WAF on internet-facing ALBs/CloudFront
```

---

## AWS Attack Paths (Red Team Perspective)

```
SSRF on EC2 app
      ↓
IMDSv1 → steal IAM role credentials
      ↓
aws sts get-caller-identity (confirm access)
      ↓
aws iam list-roles / list-policies (enumerate)
      ↓
Privilege escalation via:
  - iam:CreatePolicyVersion (add new policy version with *)
  - iam:AttachUserPolicy (attach AdministratorAccess to self)
  - iam:CreateLoginProfile (add console password to existing admin)
  - lambda:InvokeFunction + iam:PassRole (execute Lambda as admin role)
      ↓
Full account compromise → S3 exfil, EC2 pivot, secrets extraction
```

```bash
# PrivEsc check — list what current identity can do
python3 enumerate-iam.py --access-key $AWS_ACCESS_KEY_ID \
  --secret-key $AWS_SECRET_ACCESS_KEY --session-token $AWS_SESSION_TOKEN

# Pacu — AWS exploitation framework
python3 pacu.py
# Pacu commands:
# run iam__enum_permissions
# run iam__privesc_scan
# run s3__enum_bucket_names
```

---

## Key Security Services Summary

| Service | Purpose |
|---------|---------|
| **CloudTrail** | API audit log — who did what, when |
| **GuardDuty** | Threat detection — ML-based anomaly (compromised keys, recon) |
| **Security Hub** | Aggregate findings from all services + compliance scores |
| **AWS Config** | Continuous resource configuration compliance |
| **IAM Access Analyzer** | Find unintended external access to resources |
| **Macie** | S3 sensitive data discovery (PII, credentials) |
| **Inspector** | EC2/container vulnerability scanning |
| **KMS** | Key management for encryption |
| **Secrets Manager** | Store and rotate secrets (DB passwords, API keys) |
| **WAF** | Web Application Firewall for ALB / CloudFront / API Gateway |

---

## Related Notes

- [[Cloud - Container Security]] ← Docker/Kubernetes on AWS (ECS/EKS)
- [[Cloud - CI/CD Security]] ← GitHub Actions, CodePipeline
- [[Green - Instruments]] ← AWS CLI, Prowler, checkov command cheat sheet

↩ [[Green - Monographs]] · [[Green]] · [[Mind Palace]]
