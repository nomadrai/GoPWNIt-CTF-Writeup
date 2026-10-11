---
title: "Carapace"
ctf: "GoPWNIt CTF 2026"
date: 2026-10-10
category: web
difficulty: hard
points: 150
flag_format: "CTF{...}"
author: "nomadrai"
---

# Carapace

## Summary

The description emphasizes biased secrets. The retained scratch implementation
targets ECDSA-signed document capabilities and models a short-nonce Hidden
Number Problem attack, followed by a forged capability for protected key material.

## Solution — probable approach

`carapace_work.py` preserves the request format and attack hypothesis. It is
scratch code, not a retained successful execution trace. In particular, its
128-bit nonce bound remains an assumption in this account.

### 1. Collect and parse signed document capabilities

The saved code requests capabilities from `/api/v1/vault/capability` for a
document available to the current account. A capability consists of a
Base64url payload, a dot, and a Base64url signature. The signature is a raw
64-byte `r || s` pair, rather than ASN.1 DER.

Hash the exact decoded payload bytes with SHA-256 to obtain `z`. The script
uses the P-256 curve order and checks signature diversity before attempting
recovery. Repeatedly serializing parsed JSON would risk changing the signed bytes.

### 2. Exploit the hypothesized nonce bias

For each signature, ECDSA implies:

```text
k_i = (r_i/s_i)*d + z_i/s_i (mod n)
```

If the nonces really are bounded to 128 bits, these residues are small relative
to the 256-bit group order. Construct a scaled HNP lattice and apply LLL.
Check any candidate against the nonce bounds and, where available, the
service's public key before treating it as the signing scalar.

### 3. Forge a protected-document capability

The scratch script constructs claims naming document `1` and kind
`key-material`, with subject, version, issuance, expiry, and unique identifier
fields. It signs the exact JSON bytes with P-256/SHA-256, converts the signature
to raw `r || s`, and submits it to `/api/v1/vault/open` alongside `docId`.

The probable final step is to use the returned protected material as instructed
by the service to obtain the flag. A successful vault response and final
plaintext are not present in the saved record.

## Tools and lesson

The existing script uses requests, cryptography, and fpylll. Small ECDSA nonces
can compromise an otherwise correctly verified capability system. Account
tokens from the scratch file are intentionally excluded from this writeup.

## Flag / key

**Submission note (participant-provided):** The flag/key was not saved at the
moment of submission due to hurry.
