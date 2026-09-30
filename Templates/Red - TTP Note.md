---
type: ttp
room: red
tactic: "{{tactic}}"
technique_id: "T{{id}}"
technique_name: "{{name}}"
platform: [Windows, Linux, macOS]
privilege_required: user
tags: [red/{{tactic_slug}}, mitre/{{id}}]
date_added: {{date}}
last_reviewed: {{date}}
status: draft
---

# T{{id}} — {{name}}

> [!quote]
> *"When you have eliminated the impossible, whatever remains, however improbable, must be the truth."*
> — *The Sign of the Four*

**Tactic:** `{{tactic}}` | **Technique:** `T{{id}}` | **Privilege:** `{{privilege_required}}`

---

## The Theory *(What & Why)*

Brief description of the technique: what it achieves, why an adversary would choose it, and where it sits in the attack chain.

**ATT&CK Reference:** [T{{id}}](https://attack.mitre.org/techniques/T{{id}}/)

---

## The Method *(How)*

### Pre-conditions
- What access or state is required before this technique can be used?

### Execution Steps

1. Step one — what the adversary does first
2. Step two — pivot or chaining
3. Step three — confirmation of success

---

## The Instruments *(Tools)*

| Tool | Purpose | Platform |
|------|---------|----------|
| `tool-name` | Brief purpose | Linux/Windows/Both |
| `tool-name` | Brief purpose | Linux/Windows/Both |

---

## The Commands *(Standard Usage)*

### Discovery / Setup
```bash
# What to run to confirm pre-conditions or identify targets
command --flag target
```

### Primary Execution
```bash
# Core command(s) for this technique
command --flag target
```

### Windows Variant
```powershell
# PowerShell / cmd equivalent if applicable
Invoke-Command ...
```

### Verification / Confirmation
```bash
# How to confirm the technique succeeded
command --flag
```

---

## The Evidence *(Artefacts Left Behind)*

What does this technique leave on disk, in memory, in logs, or on the network?

- **Disk:** files dropped, registry keys written
- **Memory:** suspicious process trees, injected memory regions
- **Network:** traffic patterns, anomalous connections
- **Logs:** event IDs, syslog entries generated

---

## Countermeasures *(Blue's Response)*

How would Scotland Yard detect or prevent this?

| D3FEND Countermeasure | Implementation |
|----------------------|----------------|
| `D3-XXX Name` | Brief description |

> [!warning] OPSEC Considerations
> What mistakes operators commonly make with this technique that lead to detection.

---

## Sub-techniques

| Sub-ID | Name | Key Difference |
|--------|------|----------------|
| T{{id}}.001 | Sub-technique name | What makes it distinct |

---

## The Casebook *(Real-World Usage)*

- **APT Group / Campaign:** Who has used this?
- **Reference:** Link or CVE if applicable

---

## Key Deductions

> [!note] Holmes' Verdict
> One-sentence summary of when to reach for this technique and its most important gotcha.

---

## Related Notes

- [[]] ← Parent tactic note
- [[]] ← Related technique
- [[]] ← Relevant tool note

↩ [[Red]] · [[Mind Palace]]
