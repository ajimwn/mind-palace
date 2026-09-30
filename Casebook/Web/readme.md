---
type: category
room: casebook
tags: [casebook/web]
---

# Web: The Speckled Band

> [!quote]
> "I had come to an entirely erroneous conclusion, which shows, my dear Watson, how dangerous it always is to reason from insufficient data."
> — *The Adventure of the Speckled Band*

The attack came through a small opening nobody thought to guard. Web challenges are the same: find the overlooked input and see what it lets through.

## Subjects

- SQL injection
- Cross-site scripting (XSS)
- Authentication and session flaws (JWT, cookies)
- File upload and inclusion (LFI/RFI)
- Server-side vulnerabilities (SSRF, SSTI, command injection)
- Insecure deserialization and access control (IDOR)

## First Moves

1. Map the application: pages, parameters, and hidden paths.
2. Read the source, cookies, headers, and `robots.txt`.
3. Test every input for unexpected behavior.

## Instruments

Burp Suite, browser DevTools, ffuf, gobuster, sqlmap, curl, CyberChef

## Cases

Filed in this folder as `Platform - Case Name`.

↩ [[Casebook]] · [[Mind Palace]]