---
title: "scanline-seance"
ctf: "GoPWNIt"
date: 2026-10-10
category: forensics
difficulty: easy
points: 50
flag_format: "GPI{...}"
author: "nomadrai"
---

# scanline-seance

## Summary

The PNG hides a passphrase in its IDAT chunk lengths and an AES-GCM payload in its scanline filter selectors. Extracting these encoding choices recovers the flag with a valid authentication tag. Challenge by **gopwnit**, listed under **Steganography**.

## Solution

### Step 1: Inspect how the image was encoded

The attachment `1791631767090-dist.zip` contains `dist/seance.png`, a non-interlaced, 900 × 2048, 8-bit RGB PNG. The description's clue—“The picture is a lie, but the way it was written down is not”—points toward PNG encoding details.

The first 22 IDAT chunk lengths are:

```text
119 114 49 116 116 51 110 45 49 110 45 116 104 51 45 109 52 114 103 49 110 115
```

Interpreting these lengths as ASCII spells the passphrase:

```text
wr1tt3n-1n-th3-m4rg1ns
```

The final IDAT contains the remaining compressed data. Joining all IDAT contents and zlib-decompressing them produces 5,531,648 bytes. Each scanline is `1 + 900 × 3 = 2701` bytes; the leading byte selects that row's PNG filter.

Every filter selector is in `0..3`, so each represents two bits. Packing four selectors per byte, most-significant pair first, starts with `00 4d`: a two-byte big-endian payload length of **77**. The next 77 bytes contain a 12-byte nonce, 49-byte ciphertext, and 16-byte GCM tag.

The metadata comment and appended ZIP contain other flag-shaped strings. They are decoys; the authenticated scanline payload is the solution.

### Step 2: Derive the key and authenticate the flag

Use `SHA256(passphrase)` as the AES-256-GCM key. The complete solver below starts from the original ZIP and verifies the authentication tag before printing the flag. Tools: Python 3 standard library and PyCryptodome (tested with 3.24.0).

```python
import hashlib
import struct
import zipfile
import zlib
from Crypto.Cipher import AES

with zipfile.ZipFile("1791631767090-dist.zip") as archive:
    png = archive.read("dist/seance.png")

offset, idats = 8, []
while offset < len(png):
    size = struct.unpack_from(">I", png, offset)[0]
    kind = png[offset + 4:offset + 8]
    data = png[offset + 8:offset + 8 + size]
    offset += size + 12
    if kind == b"IHDR":
        width, height = struct.unpack_from(">II", data)
    elif kind == b"IDAT":
        idats.append(data)
    elif kind == b"IEND":
        break

passphrase = bytes(len(chunk) for chunk in idats[:-1])
raw = zlib.decompress(b"".join(idats))
stride = 1 + width * 3
assert len(raw) == height * stride
filters = raw[::stride]
packed = bytes(
    sum(filters[i + j] << (6 - 2 * j) for j in range(4))
    for i in range(0, len(filters), 4)
)
length = int.from_bytes(packed[:2], "big")
payload = packed[2:2 + length]
key = hashlib.sha256(passphrase).digest()
cipher = AES.new(key, AES.MODE_GCM, nonce=payload[:12])
flag = cipher.decrypt_and_verify(payload[12:-16], payload[-16:])
print(flag.decode("ascii"))
```

The saved `solve.py` also checks PNG chunk CRCs and accepts either the ZIP or extracted PNG. From the project root:

```bash
python -m pip install pycryptodome
python "scanline-seance/solve.py"
```

## Flag

```text
GPI{th3_r0ws_r3m3mb3r_h0w_th3y_w3r3_wr1tt3n_3a7c}
```
