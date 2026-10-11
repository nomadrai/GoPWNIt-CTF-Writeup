---
title: "SHAKIRA The Broken Encore"
ctf: "GoPWNIt CTF 2026"
date: 2026-10-10
category: reverse
difficulty: hard
points: 150
flag_format: "ENCORE{...}"
author: "nomadrai"
---

# SHAKIRA The Broken Encore

## Summary

The x86-64 ELF verifier expects five consecutive ten-byte fragments encoded as
100 hexadecimal characters inside `ENCORE{...}`. Invert the first stage's Feistel
network, then observe the expected values computed by the remaining stages. The
recovered flag passes all five checks in the original executable and matches its
embedded SHA-256 digest.

## Solution

### 1. Reverse the footwork transform

The supplied archive contains `encore`, `liblighting.so`, `choreography.dat`,
`rhythm.bc`, and a 224 MiB `rehearsal.vault`. Inspect the native verifier using
`strings`, `readelf`, and `objdump -d -M intel`.

The main function at ELF virtual address `0x12e0` requires a 108-character flag,
decodes five 20-character hex fragments, and checks them sequentially. Stage 1
calls the validation routine at `0x1e20`, which transforms the first fragment via
`0x1bf0` and compares it to bytes `0x90:0x9a` in `choreography.dat`:

```text
7bc8386f3a8f30eaaf8a
```

The transformation uses these constants from `encore` (ELF virtual addresses and
file offsets coincide in this region):

| Constant | Offset | Length |
|---|---:|---:|
| Four five-byte round keys | `0x58a0` | 20 |
| Byte permutation | `0x58b8` | 10 |
| AES S-box | `0x58e0` | 256 |

First substitute all ten input bytes with the S-box and permute them using
`permuted[P[i]] = SBOX[input[i]]`. Split the result into five-byte halves `L, R`
and apply four Feistel rounds:

```text
rotations = [3, 5, 2, 4, 1]
F_r(R)[i] = SBOX[((R[i] + K[5*r+i]) mod 256)
                 XOR ROL8(R[(i+1) mod 5], rotations[i])]
(L, R) = (R, L XOR F_r(R))
```

Invert rounds in reverse order with `(L, R) = (R XOR F_r(L), L)`, undo the
permutation, and apply the inverse S-box. This yields:

```text
2372f289d37d29394982
```

### 2. Observe the remaining stage outputs

The remaining checks compute their expected fragments and call `memcmp` with
length ten. Each stage depends on earlier correct fragments, so recover them
sequentially with the next fragment initially set to ten zero bytes.

`memcmp_trace.c` is an `LD_PRELOAD` observer that logs the left-hand operand of
mismatched ten-byte comparisons. It preserves the comparison's actual return
value; the verifier still rejects an incorrect candidate normally.

The call sites in `encore` are:

| Stage | Comparison call | Recovered fragment |
|---|---:|---|
| Rhythm VM | `0x1679` | `046737216602f8fd12f1` |
| Rehearsal vault | `0x1701` | `17c9af5ebe37dbcd43bc` |
| Lighting library | `0x178a` | `6a66d4fab5cd9ad24905` |
| Final authenticated record | `0x17e6` | `e3ad427e2f896afde869` |

After each execution, insert the observed fragment and run the next candidate.
This lets the supplied implementation execute its VM, vault recovery, lighting
matrix, and authenticated-record processing with the correct preceding inputs.

### 3. Verify the complete flag

The final digest check hashes the complete flag string and compares against the
32 bytes embedded at file offset `0x5880`:

```text
f56ff1122a6d5c58ced7cec85854466e22da2e9831f2f163b86eb47aabbcc762
```

The solver checks this independently using Python's `hashlib.sha256`, then runs
the original verifier again with `LD_PRELOAD` removed. It reports:

```text
[+] [CUE-FOOTWORK-01] Footwork sequence synchronized.
[+] [CUE-RHYTHM-02] Rhythm Machine VM synchronized.
[+] [CUE-VAULT-03] Rehearsal Vault chunk synchronized.
[+] [CUE-LIGHTS-04] Lighting Score matrix synchronized.
[+] [CUE-ENCORE-05] Master Encore sequence synchronized.
[+] ALL 5 REHEARSAL CUES SYNCHRONIZED!
[+] The Broken Encore has been fully restored.
```

The platform initially returned HTTP 429. Logging out and back in did not clear
the cooldown. After waiting for the indicated retry interval, the exact same
flag was accepted on **2026-10-10 at 17:23:39 UTC**:

```json
{"message": "Flag accepted.", "rank": "#25 Solve", "points": 150}
```

The current downloaded archive and the local original have identical MD5 hashes:
`bef99fa4cb9b6ee6876fb33159ab011a`.

## Reproduction

Requirements: Linux x86-64, Python 3, and GCC. The complete solver is `solve.py`,
with its comparison observer in `memcmp_trace.c`. It starts from the supplied
files, builds the observer in a temporary directory, recovers all five fragments,
and finishes with an unmodified-verifier check:

```bash
python "SHAKIRA The Broken Encore/solve.py"
```

## Flag

```text
ENCORE{2372f289d37d29394982046737216602f8fd12f117c9af5ebe37dbcd43bc6a66d4fab5cd9ad24905e3ad427e2f896afde869}
```
