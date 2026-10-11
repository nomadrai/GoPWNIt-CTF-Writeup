---
title: "Tallow"
ctf: "GoPWNIt CTF 2026"
date: 2026-10-10
category: crypto
difficulty: easy
points: 50
flag_format: "GPI{...}"
author: "nomadrai"
---

# Tallow

## Summary

Fourteen equal-length transmissions reuse an additive keystream. The retained
solver treats each byte position as a shared Caesar shift, uses the English
messages to recover the stream, and checks the dispatch flag's supplied hash.

## Solution — probable approach

This account follows [solve.py](solve.py); its final plaintext output was not
saved and the script was not run during writeup preparation.

### 1. Model the sealing operation correctly

The challenge supplies `handout/cipher.py`, `transmissions.txt`, and
`flag.sha256`. Each of the fourteen transmissions is 160 bytes. The operation
is addition modulo 256, so recovery uses subtraction:

```text
C[i,j] = P[i,j] + K[j] mod 256
P[i,j] = C[i,j] - K[j] mod 256
```

The same keystream byte affects every row at a given position. This permits
column-wise statistical analysis; XOR cancellation would model a different cipher.

### 2. Recover the stream from the prose rows

The existing solver leaves out each row in turn and counts columns that can
be made printable across all remaining rows. It uses this to identify the
non-prose outlier, then scores all 256 possible shifts per column against
English character frequencies in the thirteen prose messages.

Its saved comment records one ambiguity, `key reghster`, corrected using the
word `register` at offsets 110–117 of transmission zero. That prose crib fixes
the shared stream outside the flag itself and should improve the other rows too.

### 3. Check the dispatch seal

Decrypt all rows with the reconstructed stream, extract complete `GPI{...}`
strings, and compare their SHA-256 digests with `flag.sha256`. The existing
implementation requires exactly one digest-matching result.

## Tools and lesson

The saved solver uses Python's standard library. Stream reuse leaks relationships
between messages even when the combination operation is modular addition rather
than XOR; the supplied digest distinguishes the intended seal from other text.

## Flag

**Submission note (participant-provided):** The flag was not saved at the moment
of submission due to hurry.
