---
title: "Ragebait"
ctf: "GoPWNIt"
date: 2026-10-10
category: misc
difficulty: hard
points: 150
flag_format: "GPI{...}"
author: "nomad_aadi"
---

# Ragebait

## Summary

Both supplied executables are Windows x86-64 PE files built with Go 1.25.0.
Recovering Go function names and tracing `main.main` reveals a hardcoded
success-path flag before the distracting Feistel cipher and VM validation.

## Solution

### 1. Trace the actual success branch

The Go `.gopclntab` metadata retains function names despite stripped symbols.
In `golev_linux_amd64`, `main.main` starts at `0x1400ba9c0`:

- It rejects a table of 15 explicitly designated decoy flags.
- At `0x1400bac49`, it checks for a 40-byte input.
- At `0x1400bac53`, it loads a string from `0x1400feed6` and calls
  `runtime.memequal` for all 40 bytes.
- On equality it calls `GetBinaryRuntimeHash`. A nil error leads directly to
  the success block at `0x1400bb259`.
- That block prints `TRUE GOD TIER ACHIEVED: STAGE 2 CONQUERED`, then
  `OFFICIAL TOURNAMENT FLAG: %s` using the accepted input.

The self-hash is SHA-256 of the first 4096 executable bytes followed by
`go1.25.0`. Its value is displayed, but is **not compared against an expected
digest**. Only successful computation is required.

This complete extraction script starts from the original attachment:

```python
import struct
import zipfile
import pefile

with zipfile.ZipFile("1791635369181-dist.zip") as z:
    raw = z.read("dist/golev_linux_amd64")
pe = pefile.PE(data=raw)
image = pe.get_memory_mapped_image()
lea_rva = 0xbac53
assert image[lea_rva:lea_rva+3] == bytes.fromhex("488d1d")
displacement = struct.unpack_from("<i", image, lea_rva + 3)[0]
string_rva = lea_rva + 7 + displacement
flag = image[string_rva:string_rva+40].decode()
assert flag.startswith("GPI{") and flag.endswith("}")
print(flag)
```

Install the extraction dependency with `python3 -m pip install pefile`.

### 2. Verify the compiled acceptance path

`solve.py`, with its helper `analyze.py`, emulates the original machine code
using Unicorn. It supplies OS file reads and Go heap allocations, while
executing the actual decoy comparisons, final equality comparison, and
self-hash code. The resulting digest is independently checked using Python's
`hashlib`, and execution must reach the genuine success branch.

```bash
python3 -m pip install pefile capstone unicorn
python3 solve.py
```

Verified output:

```text
golev_linux_amd64: compiled acceptance branch reached; self-hash 62f1654ca5ca11225fb42235b817fe55f4d8a5e4128197f80f18f39d3b69d960
golev_windows_amd64.exe: compiled acceptance branch reached; self-hash 6411a82c028566289c9b2896e69a2f8d329e154794e72fd732b365b682868df4

FLAG: GPI{b3y0nd_41_l0g1c_r3c0nstruct10n_9f82}
```

The flag occurs once in each executable and is absent from the decoy table.
Validation was local machine-code emulation; no scoreboard submission was made.

## Flag

```text
GPI{b3y0nd_41_l0g1c_r3c0nstruct10n_9f82}
```
