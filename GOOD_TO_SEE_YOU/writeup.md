---
title: "GOOD_TO_SEE_YOU"
ctf: "GoPWNIt CTF 2026"
date: 2026-10-10
category: forensics
difficulty: medium
points: 100
flag_format: "GPI{...}"
author: "nomadrai"
---

# GOOD_TO_SEE_YOU

## Summary

The attachment contains an ext4 disk image from a departing employee's machine.
Although the archive was deleted, file carving recovered its ZIP contents and
the flag.

## Solution

### 1. Carve the deleted archive

The original attachment is `1791619786594-player_package.zip`; its extracted
`disk.img` is 31,457,280 bytes. The retained Foremost audit records this exact
competition-time invocation:

```bash
foremost -i disk.img -o carved -t zip
```

Foremost 1.5.7 identified one 220-byte ZIP at byte offset **6,230,016** and saved
it as `carved/zip/00012168.zip`. Its filename uses 512-byte block units:
`12168 * 512 = 6230016`. The audit records completion on 10 October 2026.

### 2. Read the recovered content

The recovered archive and its extracted `flag.txt` both remain in `carved/zip/`.
The text identifies itself as the real flag. The challenge narrative mentions
an encrypted archive and deleted shell history, but the retained record only
establishes the carving and final extraction; it does not preserve a password
recovery step.

For reproduction on a fresh working copy, carve ZIP structures from the image,
inspect the recovered archive's directory and encryption flags, and extract
its contents. The original image need not be mounted or modified.

## Evidence and tools

- [carved/audit.txt](carved/audit.txt): exact invocation, offset, size, and output.
- [carved/zip/00012168.zip](carved/zip/00012168.zip): recovered archive.
- [carved/zip/flag.txt](carved/zip/flag.txt): saved plaintext.
- Tools: Foremost 1.5.7 and a ZIP reader.

Deleting a filesystem entry does not necessarily overwrite its data blocks;
format-aware carving can recover a small intact archive without its original name.

## Flag

```text
GPI{z1p_c4rv1ng_1s_fun_9ae865d44a6f}
```
