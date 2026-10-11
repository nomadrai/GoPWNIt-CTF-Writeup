---
title: "Fouriers-ghost"
ctf: "GoPWNIt CTF 2026"
date: 2026-10-10
category: forensics
platform_category: "Steganography"
difficulty: easy
points: 50
flag_format: "GPI{...}"
author: "nomadrai"
---

# Fouriers-ghost

## Summary

A sixteen-bit PNG splits its clues across two byte planes. A Fourier transform
of the low byte reveals instructions for extracting an authenticated encrypted
payload from the high byte.

## Solution

### 1. Separate the image into byte planes

The description says that only half of every pixel is visible. The saved
[solve.py](solve.py) loads `dist/ghost.png` as unsigned sixteen-bit samples and
separates `low = pixels & 255` from `high = pixels >> 8`.

Subtract the low plane's mean, take its two-dimensional FFT, center the spectrum
with `fftshift`, and display `log1p(abs(spectrum))`. Percentile scaling makes
the frequency-domain lettering visible. The retained [spectrum.png](spectrum.png)
and the solver's transcription record:

```text
HIGH BYTE, bit 0
columns right to left, each column bottom-up
u16be L | nonce[12] | AES-256-GCM
key = SHA256(PASS)
PASS = MERIDIAN-204-LOWTIDE
```

### 2. Extract and authenticate the payload

Read bit zero of the high plane in the specified order. In NumPy the traversal
is `(high[::-1, ::-1].T & 1).reshape(-1)`. Pack bits most-significant first.
Interpret the first two bytes as a big-endian length and take that many bytes
after the header. The first twelve payload bytes are the GCM nonce; the rest
are ciphertext followed by the authentication tag.

Use the raw SHA-256 digest of `MERIDIAN-204-LOWTIDE` as the AES-256 key, with no
associated data. The saved script performs authenticated decryption and writes
`flag.txt`; that output is already present locally.

## Reproduction reference

The complete existing solver requires Python 3, NumPy, Pillow, and
`cryptography`. From the repository root:

```bash
python3 "Fouriers-ghost/solve.py"
```

This command documents the retained solution; it was not rerun for this handoff.
The important distinction is between visual pixel values and information encoded
in the frequency domain of a separate bit-depth plane.

## Flag

Source: [flag.txt](flag.txt).

```text
GPI{l0w_byt3s_dr34m_1n_fr3qu3ncy_sp4c3_2c96}
```
