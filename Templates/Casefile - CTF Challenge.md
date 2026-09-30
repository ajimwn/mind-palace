---
type: casefile
case: "{{challenge_name}}"
platform: "{{platform}}"   # picoCTF / TJCTF / HTB / THM / CTF
category: "{{category}}"   # Cryptography / Forensics / Pwn / Rev / Web / OSINT / Steganography / Misc
difficulty: "{{difficulty}}"
event: "{{event_name}}"
date: {{date}}
points: {{points}}
status: solved  # solved / unsolved / partial
tags: [casebook, "{{platform_slug}}", "{{category_slug}}"]
---

# {{challenge_name}}

> [!quote]
> *"There is nothing more deceptive than an obvious fact."*
> — *The Boscombe Valley Mystery*

**Platform:** `{{platform}}` | **Event:** `{{event_name}}` | **Category:** `{{category}}` | **Difficulty:** `{{difficulty}}` | **Points:** `{{points}}`

---

## The Mystery *(Challenge Description)*

> Paste the original challenge prompt here.

**Files provided:**
- `filename.ext` — description

---

## First Observations

What did you notice immediately upon looking at the challenge? Initial impressions, obvious clues, file type identification.

```bash
file challenge_file
xxd challenge_file | head -20
strings challenge_file | head -50
exiftool challenge_file   # if image
```

---

## The Investigation *(Steps Taken)*

### Step 1 — [Observation / Tool]
```bash
# Command and output
```
**What it revealed:**

### Step 2 — [Deduction / Pivot]
```bash
# Command and output
```
**What it revealed:**

### Step 3 — [Exploitation / Solution]
```bash
# Final command / script
```

---

## The Resolution *(Solution)*

**Flag:** `[REDACTED — record privately below]`

```
flag{PRIVATE — do not commit if publishing}
```

**Solution Summary in one sentence:**

---

## What Watson Missed *(Red Herrings & Mistakes)*

- Spent X minutes on Y because Z — it was a dead end because…
- Almost missed: the key clue was actually…

---

## Lessons Learned

> [!note] Key Takeaways
> - Main technique: describe it
> - Tool discovered or used effectively: [[link if note exists]]
> - Would apply to: [[related TTP or casebook note]]

**Category Pattern Recognised:**
This challenge follows the `[pattern name]` pattern common in `[category]` challenges. Related: [[]]

---

## Solve Script (if applicable)

```python
#!/usr/bin/env python3
# Solve script for {{challenge_name}}
# {{platform}} — {{event_name}}

# ...

print(f"Flag: flag{{...}}")
```

---

## References

- Challenge: {{platform_url}}
- Tool used: [[]]
- Writeup inspiration: {{reference_url}}

↩ [[Casebook/Casebook]]
