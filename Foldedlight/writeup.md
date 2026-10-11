---
title: "Foldedlight"
ctf: "GoPWNIt"
date: 2026-10-10
category: crypto
difficulty: medium
points: 100
flag_format: "GPI{...}"
author: "nomadrai"
---

# Foldedlight

## Summary

Nonce reuse exposes a damaged RSA private key through a known-plaintext attack.
Parsing its DER integers directly reveals an intact prime factor, allowing the
private key to be reconstructed and the RSA-encrypted dispatch to be decrypted.

## Solution

### Step 1: Recover and repair the private key

`sealer.py` XORs each member with a SHA-256-derived keystream, using
`nonce = index % 3`. Members 00 and 03 therefore share their keystream.
The supplied `licence.txt` is member 00's full plaintext, so:

```text
plaintext_03 = ciphertext_03 XOR ciphertext_00 XOR licence
```

The recovered member is a PEM-encoded PKCS#1 RSA private key. Its DER structure
is intact, but five bytes of `d` and four bytes of `q` were replaced with zero.
Directly parsing the ASN.1 INTEGER fields bypasses the consistency checks that
would make a normal key-loading library reject it.

The stored `p` divides the stored modulus `n` exactly. Recover `q = n // p`,
then compute `d = e^-1 mod lcm(p-1, q-1)`. The reconstructed key agrees with
all three surviving CRT parameters (`dp`, `dq`, and `qinv`).

### Step 2: Decrypt and verify the dispatch

`dispatch.enc` contains an unpadded RSA ciphertext written as hexadecimal.
Compute `m = c^d mod n` and decode it as big-endian bytes. Raising the recovered
message to `e` modulo `n` reproduces the supplied ciphertext.

The following complete solver runs from the `Foldedlight` directory and reads
the original attachment directly. It requires only Python 3.9+ and its standard
library.

```python
import base64
import math
from zipfile import ZipFile


def tlv(data, offset=0):
    tag, length = data[offset:offset + 2]
    offset += 2
    if length & 0x80:
        size = length & 0x7f
        length = int.from_bytes(data[offset:offset + size], "big")
        offset += size
    return tag, data[offset:offset + length], offset + length


with ZipFile("1791624363995-handout.zip") as archive:
    licence = archive.read("handout/licence.txt")
    c0 = archive.read("handout/archive/member-00.bin")
    c3 = archive.read("handout/archive/member-03.bin")
    pem = bytes(a ^ b ^ c for a, b, c in zip(licence, c0, c3))
    ciphertext = int(archive.read("handout/dispatch.enc").strip(), 16)

der = base64.b64decode(b"".join(pem.splitlines()[1:-1]), validate=True)
tag, sequence, end = tlv(der)
assert tag == 0x30 and end == len(der)
values, offset = [], 0
while offset < len(sequence):
    tag, value, offset = tlv(sequence, offset)
    assert tag == 2
    values.append(int.from_bytes(value, "big"))

version, n, e, damaged_d, p, damaged_q, dp, dq, qinv = values
assert version == 0 and 1 < p < n and n % p == 0
q = n // p
d = pow(e, -1, math.lcm(p - 1, q - 1))
assert (d % (p - 1), d % (q - 1), pow(q, -1, p)) == (dp, dq, qinv)
message = pow(ciphertext, d, n)
assert pow(message, e, n) == ciphertext
print(message.to_bytes((message.bit_length() + 7) // 8, "big").decode())
```

The saved `solve.py` additionally verifies the exact nine-byte corruption.
Run it from the repository root after extracting the handout:

```bash
unzip -n "Foldedlight/1791624363995-handout.zip" -d "Foldedlight"
python3 "Foldedlight/solve.py"
```

Verified output:

```text
Repaired d: 5 zeroed bytes at offsets [64, 65, 66, 67, 68]
Repaired q: 4 zeroed bytes at offsets [31, 32, 33, 34]
Verified: p*q=n, all CRT fields, and flag re-encryption
GPI{th3_l1br4ry_r3fus3d_s0_1_p4rs3d_1t_mys3lf}
```

## Flag

```text
GPI{th3_l1br4ry_r3fus3d_s0_1_p4rs3d_1t_mys3lf}
```
