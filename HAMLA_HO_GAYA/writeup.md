---
title: "HAMLA_HO_GAYA"
ctf: "GoPWNIt CTF 2026"
date: 2026-10-10
category: forensics
difficulty: medium
points: 100
flag_format: "FLAG{...}"
author: "nomadrai"
---

# HAMLA_HO_GAYA

## Summary

The challenge supplies logs, a packet capture, and a memory image for
cross-artifact investigation. The retained memory-string extraction contains
the flag directly, while the logs provide context for suspicious SSH activity.

## Solution

### 1. Identify the relevant incident context

The extracted files are `logs.txt`, `traffic.pcap`, and `memory.mem`.
`logs.txt` contains a cluster of SSH login failures for account `backup_adm`
from `10.0.0.5`, surrounded by repetitive health-check and IIS entries.

The SSH entries explicitly use timezone `+03:00`, whereas the surrounding
entries use `UTC+00:00`. Any timeline correlation must convert these timestamps
to one timezone before matching them with packet or memory evidence. An offset
difference alone does not establish clock tampering.

### 2. Locate the plaintext retained in memory

A pre-existing strings extraction, `memory_strings.txt`, contains this complete
line at line **65046**:

```text
FLAG{m3m0ry_f0r3n51c5_m45t3r}
```

This is already saved plaintext, so the writeup can cite it without repeating
memory analysis. The record supports direct memory-string recovery; it does not
preserve a packet-by-packet exploit narrative or a scoreboard acceptance receipt.

## Reproduction reference

The corresponding offline inspection would use a strings utility to extract
printable memory content, then locate the advertised flag wrapper. Wireshark
or tshark and a text viewer are the supporting tools for correlating the
network activity and normalized log timestamps.

The main lesson is to inspect simple retained plaintext before assuming a
memory image requires a full operating-system profile or a complex decryption
chain. The flag below is transcribed from the existing extraction.

## Flag

```text
FLAG{m3m0ry_f0r3n51c5_m45t3r}
```
