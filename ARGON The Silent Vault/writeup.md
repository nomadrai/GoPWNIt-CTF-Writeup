---
title: "ARGON The Silent Vault"
ctf: "GoPWNIt CTF 2026"
date: 2026-10-10
category: crypto
difficulty: hard
points: 150
flag_format: "GPI{...}"
author: "nomadrai"
---

# ARGON The Silent Vault

## Summary

The signing appliance leaks the upper twelve bits of ECDSA nonces. A Hidden
Number Problem attack recovers its signing scalar, which opens an RSA recovery
bundle; a common-modulus attack then supplies the token needed for the final
Argon2id-protected vault.

## Evidence

The local `extracted/` directory retains the public specification, signatures,
diagnostics, recovered scalar, decrypted `recovery.json`, `token.bin`, and
`vault.txt`. The explanation below follows that specification and those saved
intermediate results. The original lattice invocation is not retained.

## Solution

### 1. Recover the signing scalar from partial nonces

`public.json` specifies 96 secp256k1 signatures. Each twenty-byte diagnostic
record contains a sixteen-byte signature identifier and two big-endian
sixteen-bit fields. Match records to `signatures.jsonl` by identifier and select
channel `0x0042`. Its value `h_i` gives:

```text
k_i = h_i * 2^244 + u_i,  0 <= u_i < 2^244
s_i*k_i = z_i + r_i*d (mod n)
u_i = (r_i/s_i)*d + z_i/s_i - h_i*2^244 (mod n)
```

Here division means multiplication by a modular inverse, and `z_i` is the
SHA-256 digest of the ASCII message reduced modulo the curve order. These
bounded modular residues form an HNP lattice. Center the unknown intervals,
reduce the lattice, and check a candidate scalar against the public point `Q=dG`
and the leaked intervals.

The scalar already saved in `extracted/recovered_d.txt` is:

```text
108015446956611248105191314581707945480675104741141439159246043356998594594060
```

### 2. Open the recovery bundle and recover its token

Derive a 32-byte wrapping key with HKDF-SHA256 over `d.to_bytes(32, "big")`,
the public `wrap_salt`, and info `ARGON/RECOVERY/v1`. `recovery.enc` stores
`nonce[12] | tag[16] | ciphertext`; its AES-GCM associated data is
`ARGON/RECOVERY/v1`. Libraries expecting ciphertext followed by tag require
reordering that envelope.

The saved plaintext `recovery.json` contains two encryptions of the same token
under one RSA modulus, with coprime exponents `65537` and `100003`. Extended
Euclid gives integers `a,b` such that `a*e1+b*e2=1`, so:

```text
m = c1^a * c2^b mod N
T = m.to_bytes(32, "big")
```

Negative exponents require modular inverses. The recovered token is retained
as `extracted/token.bin`.

### 3. Derive the final vault key

Compute `P = SHA256(b"ARGON/FINAL/v1" + d_bytes + T)`. Apply Argon2id version
19 with the public final salt, memory cost 65,536 KiB, time cost 3, parallelism
4, and output length 32. Open `vault.enc` using the same nonce/tag/ciphertext
layout and associated data `ARGON/VAULT/v1`.

The resulting plaintext is retained in [extracted/vault.txt](extracted/vault.txt).
Tools for this path are Python, an HNP-capable lattice library, HKDF/AES-GCM,
and Argon2id. The key lesson is that small per-signature leaks accumulate, and
strong final encryption cannot repair an exposed upstream key.

## Flag

```text
GPI{argon_d5e6376be1105fffc0e40e9ffcf08f52}
```
