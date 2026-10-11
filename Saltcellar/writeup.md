---
title: "Saltcellar"
ctf: "GoPWNIt CTF 2026"
date: 2026-10-10
category: crypto
difficulty: medium
points: 100
flag_format: "GPI{...}"
author: "nomadrai"
---

# Saltcellar

## Summary

ECDSA nonces are generated as a fixed public 64-bit step multiplied by a
192-bit random integer. Although the resulting nonces can span 256 bits,
modular division by the step exposes small hidden values. An eight-signature
Hidden Number Problem lattice recovers the private key, which decrypts the flag.

## Solution

### 1. Normalize the signature equations

The signer uses secp256k1 and generates:

```python
STEP = 0x9E3779B97F4A7C15
k = STEP * secrets.randbits(192)
```

For each signature, ECDSA gives:

```text
s_i * k_i = h_i + r_i*d (mod n)
```

Write `k_i = STEP*u_i`, where `0 < u_i < B = 2^192`. Since `s_i*STEP`
is invertible modulo the prime curve order `n`, define:

```text
a_i = r_i * inverse(s_i*STEP, n) mod n
b_i = h_i * inverse(s_i*STEP, n) mod n
u_i = a_i*d + b_i (mod n)
```

These are modular linear equations sharing the unknown key `d`, with bounded
192-bit residues. The weakness is a known multiplicative factor, not zero low
bits or a bit shift: `STEP` is odd.

### 2. Recover the key with LLL

Use the first eight signatures and this ten-dimensional integer row basis:

```text
[ n²   0   ...   0      0      0  ]
[  0  n²   ...   0      0      0  ]
[             ...                ]
[  0   0   ...  n²      0      0  ]
[ na₀ na₁  ... na₇      B      0  ]
[ nb₀ nb₁  ... nb₇      0     Bn  ]
```

A combination of these rows produces a short vector of the form:

```text
(n*u_0, ..., n*u_7, B*d', B*n), where d' = d (mod n).
```

Run `fpylll.LLL.reduction`, find rows whose final coordinate is `±B*n`,
normalize the sign, and recover `d` from the penultimate coordinate divided
by `B`, modulo `n`. Validate candidates against the supplied public key.

Recovered private key:

```text
5005554fa55b9e811a71e69c7caa0ad95768469ef5e8b363b363b2a77dfddc2f
```

The solver additionally reconstructs every nonce, checks its step alignment and
192-bit quotient bound, verifies `x(k_i*G) mod n == r_i`, and verifies the ECDSA
equation for all forty signatures.

### 3. Decrypt and verify

As specified in `notes.md`, form the repeated XOR mask:

```python
mask = hashlib.sha256(d.to_bytes(32, "big")).digest()
plaintext = bytes(c ^ mask[i % 32] for i, c in enumerate(ciphertext))
```

The adjacent `solve.py` performs the complete attack from the original ZIP,
including public-key and signature verification. Run from the project root:

```bash
uv run --no-project --with fpylll --with cysignals --with cryptography python Saltcellar/solve.py
```

The platform accepted the flag on **2026-10-10 at 17:53:25 UTC**:

```json
{"message": "Flag accepted.", "rank": "#27 Solve", "points": 100}
```

## Flag

```text
GPI{th3_b14s_w4s_4_mult1pl3_n0t_4_sh1ft}
```
