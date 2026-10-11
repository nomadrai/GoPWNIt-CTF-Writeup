---
title: "leaky crt"
ctf: "GoPWNIt CTF 2026"
date: 2026-10-10
category: crypto
difficulty: easy
points: 50
flag_format: "GPI{...}"
author: "nomadrai"
---

# leaky crt

## Summary

A debug log exposes RSA's private CRT exponent `dp`. The retained solver uses
that value and the small public exponent to factor the modulus, reconstruct
the private exponent, and decrypt the supplied ciphertext.

## Solution — probable approach

This explanation follows the existing [solve.py](solve.py). Its output was not
saved locally, so no factoring or decryption was rerun for this handoff.

### 1. Convert the CRT leak into a factor relation

Read `n`, `e`, and `dp` from
`1791632272876-dist/dist/key.txt`, and `c` from the neighboring `cipher.txt`.
For the canonical CRT exponent:

```text
dp = d mod (p-1)
e*dp - 1 = k*(p-1), where 1 <= k < e
```

Enumerate the small multiplier `k`. Whenever it divides `e*dp-1`, form
`p = (e*dp-1)//k + 1` and test whether `1 < p < n` and `p` divides `n`.
An exact divisor yields the other prime as `q=n//p`.

### 2. Reconstruct and verify the plaintext

Compute `d = inverse(e, (p-1)*(q-1))`, then `m = pow(c,d,n)`. Convert `m`
to its minimal big-endian byte representation. The saved script checks all of
the following before printing a result:

- `p*q == n`;
- the reconstructed exponent agrees with the leaked `dp`;
- re-encryption reproduces `c`;
- the decoded text has the expected flag wrapper.

## Tools and lesson

The retained implementation needs only Python 3 with modular-inverse support.
A CRT exponent is enough private information to compromise RSA, even when the
full exponent and prime factors never appear in the debug log.

## Flag

**Submission note (participant-provided):** The flag was not saved at the moment
of submission due to hurry.
