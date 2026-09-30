---
type: room
room: casebook
tags: [casebook]
---

# Casebook

> [!quote]
> *"When you have eliminated the impossible, whatever remains, however improbable, must be the truth."*
> — *The Sign of the Four*

Every CTF challenge, TryHackMe room, HackTheBox machine, and lab exercise is a case. The Casebook is the permanent record of each investigation: what was observed, how it was deduced, and what was ultimately learned.

**A case poorly documented is a case half-solved.**

---

## Case Categories

| Category | Identity | Covers |
|----------|---------|--------|
| [[Casebook/Cryptography/readme\|Cryptography]] | The Dancing Men | Ciphers, encryption, hashing, encoding challenges |
| [[Casebook/Forensics/readme\|Forensics]] | The Magnifying Glass | Disk, memory, network, file analysis |
| [[Casebook/Miscellaneous/readme\|Miscellaneous]] | The Curious Incident | Puzzles, scripting, jails, and oddities |
| [[Casebook/OSINT/readme\|OSINT]] | The Commonplace Book | Open-source intelligence and geolocation |
| [[Casebook/Pwn (Binary Exploitation)/readme\|Pwn]] | The Burglar | Buffer overflows, ROP, format strings, heap |
| [[Casebook/Reverse Engineering/readme\|Reverse Engineering]] | Reasoning Backward | Disassembly, decompilation, crackmes |
| [[Casebook/Steganography/readme\|Steganography]] | The Blue Carbuncle | Data hidden in images, audio, and files |
| [[Casebook/Web/readme\|Web]] | The Locked Room | Web exploitation, API security |

---

## Filing a Case

1. Create a new note in the matching category folder.
2. Name it `Platform - Machine/Challenge Name` (e.g. `THM - Blue`, `HTB - Lame`, `picoCTF - Caesar`).
3. Use the appropriate template:
   - 🔴 **Red machine (pentest-style):** [[Templates/Casefile - Machine]]
   - 🎯 **CTF Challenge (single flag):** [[Templates/Casefile - CTF Challenge]]

---

## Platform Prefixes

| Prefix | Platform |
|--------|---------|
| `THM -` | TryHackMe |
| `HTB -` | HackTheBox |
| `PG -` | Proving Grounds (Offensive Security) |
| `VH -` | VulnHub |
| `CTF -` | Generic CTF event |
| `picoCTF -` | picoCTF |
| `TJCTF -` | TJCTF |

---

## Statistics

> [!tip] Progress Tracking
> Use a Dataview query (requires the Dataview plugin) to auto-generate this table:
> ````
> ```dataview
> TABLE platform, category, difficulty, status
> FROM "Casebook"
> WHERE type = "casefile"
> SORT date DESC
> ```
> ````

---

## Ethics and Disclosure

- Publish writeups only for **retired machines** and **concluded CTF events**.
- **Never include raw flags** in notes that may be published.
- Share **methodology and reasoning**, not spoilers.
- When in doubt, keep the repo private (it already is).

↩ [[Mind Palace]]