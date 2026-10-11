---
title: "Candle Wright"
ctf: "GoPWNIt CTF 2026"
date: 2026-10-10
category: crypto
difficulty: easy
points: 50
flag_format: "GPI{...}"
author: "nomadrai"
---

# Candle Wright

## Summary

The handout contains a Base64-wrapped sealed memo and a hash-based flag
checker. The most probable route is to recover the memo's repeating-key
obfuscation, then distinguish its actual dispatch flag from an operator-code decoy.

## Solution — probable approach

This is an inferred process based on `handout/intercept.txt` and
`handout/check.py`. No decoding, key search, or checker execution was performed
while preparing this account; the precise sealing algorithm is not documented
in a retained solve script.

### 1. Remove the transport encoding and identify the repeating structure

Discard comment and blank lines from `intercept.txt`, concatenate the remaining
Base64 lines, and decode them into sealed bytes. A repeating-key XOR layer is
a reasonable first hypothesis for this single, relatively long internal memo.
Estimate candidate periods from repeated byte patterns or normalized Hamming
distances, then score each transposed key column against English text.

The hypothesis would need validation against coherent plaintext throughout
the memo, not just a flag-shaped substring. Expected memo language can refine
ambiguous key bytes without guessing the final flag.

### 2. Read the complete memo and use the intended checker branch

`check.py` hashes the exact stripped candidate with SHA-256. It explicitly
distinguishes the dispatch flag from a second value called `OPERATOR_CODE`.
The latter produces a rejection explaining that the memo identifies it twice
as the wrong answer.

The intended final check is a match to the `REAL` digest:

```text
dcf53b17fdc8e200a01c6c4fdaba1c0ac84fc2c3f0aa09181ddf82ecf7450e83
```

That digest is a verification value, not the flag or an encryption key.

## Tools and lesson

The probable workflow uses Python's Base64 and statistical byte-analysis tools,
followed by the supplied SHA-256 checker. Reading context matters when a memo
contains both an operational code and the requested dispatch seal.

## Flag

**Submission note (participant-provided):** The flag was not saved at the moment
of submission due to hurry.
