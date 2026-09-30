---
type: ir-playbook
room: blue
phase: "{{phase}}"
incident_type: "{{incident_type}}"
severity: P1 / P2 / P3 / P4
tags: [blue/ir, blue/{{phase_slug}}, blue/{{incident_type_slug}}]
date_created: {{date}}
date_activated: 
lead:
---

# IR Playbook — {{incident_type}}

> [!quote]
> *"Come, Watson, the game is afoot. Not a word. Into your clothes and come!"*
> — *The Adventure of the Abbey Grange*

**Phase:** `{{phase}}` | **Incident Type:** `{{incident_type}}` | **Severity:** `{{severity}}`

---

## Incident Summary

> Fill this in as soon as the incident is declared.

| Field | Value |
|-------|-------|
| **Detection Time** | |
| **Declaration Time** | |
| **Affected System(s)** | |
| **Affected User(s)** | |
| **Initial Vector** | |
| **Suspected Actor** | |
| **Scope** | |

---

## Phase 1 — Prepare *(Pre-incident)*

> [!note] This section is filled before an incident occurs — part of your posture.
> - [ ] Asset inventory is current
> - [ ] Detection rules are active for this incident type
> - [ ] IR contacts list is maintained
> - [ ] Evidence preservation procedure is understood
> - [ ] Playbook has been reviewed in the last 90 days

---

## Phase 2 — Detect & Analyse

### Initial Indicators

```
IOC Type    | Value           | Source
-----------|-----------------|--------
IP          | x.x.x.x         | SIEM alert
Hash (SHA256)| abc123...      | EDR
Domain      | evil.domain.com | DNS logs
```

### Triage Checklist

- [ ] Confirm the alert is not a false positive
- [ ] Identify affected host(s)
- [ ] Identify affected user account(s)
- [ ] Determine approximate time of initial compromise
- [ ] Assess current attacker dwell time
- [ ] Determine if data was accessed or exfiltrated
- [ ] Search for lateral movement indicators

### Key Queries (adapt to your SIEM)

```
# Replace with relevant queries for this incident type
index=* source_ip="<IOC_IP>" | table _time, host, user, action
```

---

## Phase 3 — Contain

### Immediate Actions (first 30 minutes)

- [ ] Isolate affected host(s) from network (quarantine in EDR, or firewall rule)
- [ ] Disable compromised user account(s)
- [ ] Block known-bad IPs/domains at firewall and proxy
- [ ] Preserve evidence: capture memory dump, disk image if warranted
- [ ] Notify stakeholders per escalation matrix

### Evidence Preservation

```bash
# Memory dump (Windows)
.\winpmem_mini_x64.exe memory.aff4

# Memory dump (Linux)
sudo avml /path/to/memory.lime

# Disk image (Linux, forensic copy)
sudo dd if=/dev/sda of=/evidence/disk.img bs=4M status=progress
sudo sha256sum /evidence/disk.img > /evidence/disk.img.sha256

# Windows: collect artifacts with CyLR
.\CyLR.exe -o C:\Evidence
```

---

## Phase 4 — Eradicate

- [ ] Remove malware / implant from all affected hosts
- [ ] Delete persistence mechanisms (scheduled tasks, registry keys, services)
- [ ] Revoke compromised credentials and rotate passwords
- [ ] Patch exploited vulnerability
- [ ] Harden the exploited attack vector
- [ ] Scan for related malware across the environment

---

## Phase 5 — Recover

- [ ] Restore affected systems from clean backup (verify backup integrity first)
- [ ] Re-enable user accounts after password reset
- [ ] Monitor affected systems intensively for 72 hours post-recovery
- [ ] Confirm business operations are restored
- [ ] Remove temporary containment measures (firewall rules, blocks)

---

## Phase 6 — Post-Incident Review

### Timeline

| Time (UTC) | Event | Source |
|-----------|-------|--------|
| T+0 | Initial detection | |
| T+X | Containment action | |
| T+X | Eradication complete | |
| T+X | Recovery confirmed | |

### Root Cause

What was the root cause? What control failed or was missing?

### Lessons Learned

What Watson Missed — mistakes, gaps, slow decisions.

### Improvements

| Action Item | Owner | Due Date |
|-------------|-------|----------|
| | | |

---

## ATT&CK Mapping of Observed TTPs

| Technique ID | Name | Observed Artefact |
|-------------|------|------------------|
| T{{id}} | | |

---

## Related Notes

- [[IR - Detect and Analyse]]
- [[IR - Contain]]
- [[Blue - Instruments]]

↩ [[Blue - Notes]] · [[Blue]] · [[Mind Palace]]
