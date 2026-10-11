---
title: "The Hollow"
ctf: "GoPWNIt CTF 2026"
date: 2026-10-10
category: crypto
difficulty: hard
points: 150
flag_format: "GPI{...}"
author: "nomadrai"
---

# The Hollow

## Summary

Nonce reuse in a GHASH-like MAC reveals its evaluation key. That same key
decrypts the vault parameters, whose partially smooth group order permits
recovery of the vault exponent with subgroup discrete logarithms and CRT.

Challenge author: **gopwnit**.

## Solution

### 1. Recover the fold key from reused-nonce tags

All three ciphertexts are exactly 16 bytes and share the nonce
`5468654e6f6e63655265757365`. For a single ciphertext block, the MAC is

```text
T = C·H² XOR L·H XOR S
```

The common nonce makes `S` identical, while equal ciphertext lengths make `L`
identical. Subtracting two tags in characteristic two gives

```text
H² = (T₀ XOR T₁) / (C₀ XOR C₁)
H  = (H²)^(2^127)
```

Implement field multiplication exactly as specified in NIST SP 800-38D:
the integer `1 << 127` represents the multiplicative identity, and the
right-shift reduction constant is `0xe1000000000000000000000000000000`.

Both independent envelope pairs yield:

```text
H = d59d9939a0820f4dde7417d787839399
```

Recomputing every complete tag confirms the key. Decrypting the envelopes gives:

```text
release batch 07
hold batch 07!!!
amend batch 07 x
```

### 2. Recover the vault exponent

Decrypt `stage2.bin` with the counter stream
`SHA256(SHA256(H_bytes) || counter_be32)`, starting at counter zero. The result
contains a 500-bit prime `p`, generator `g = 2`, and public value `y`.

Trial division and Pollard–Brent factor `p-1` into:

```text
2 · 2039 · 4049 · 9203 · 10847 · 12413 · 21521 · 28813
· 37549 · 39821 · 43577 · 44687 · 48989 · 50651 · 50929
· 54091 · 57251 · 136973113 · 754799435023
· 1228410153261862865464230587769556123390808322427078055105669
```

The final factor is too large for a generic discrete-log attack, but it is
unnecessary. For each smaller factor `q`, use baby-step giant-step to solve:

```text
(g^((p-1)/q))^x = y^((p-1)/q) mod p
```

Combining these residues with CRT recovers `x` modulo the following 300-bit
product:

```text
1494643216714104040909347553646151362760147057897129505818431744353531397383478986304505482
```

The least nonnegative representative already satisfies `pow(g, x, p) == y`:

```text
x = 227862127400457689541605767043391334023331758700378197292470935765364360301199523982608470
```

### 3. Decrypt and submit

Encode `x` in its minimum number of big-endian bytes. Derive
`seed = SHA256(H_bytes || x_bytes)`, then XOR the hex-decoded `flag.enc` with
`SHA256(seed || counter_be32)` blocks, beginning at zero.

The complete solver is [solve.py](solve.py). It starts from the original
handout, validates all three tags and the recovered public exponent, and
decrypts the flag. Only Python 3's standard library is required:

```bash
python "The Hollow/solve.py"
```

## Flag

```text
GPI{0n3_f0ld_k3y_tw0_j0bs_4nd_4_sm00th_pr1m3}
```

The platform returned **“Flag accepted.”**, **150 points**, and **#21 Solve**
at `2026-10-10T17:57:35.828Z`. The challenge was independently confirmed in the
account's solved list. See [submission_result.json](submission_result.json).
