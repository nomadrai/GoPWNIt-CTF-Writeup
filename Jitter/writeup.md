---
title: "Jitter"
ctf: "GoPWNIt"
date: 2026-10-10
category: forensics
difficulty: medium
points: 100
flag_format: "GPI{...}"
author: "nomadrai"
---

# Jitter

## Summary

The ping intervals encode instructions for a second covert channel in the IPv4
identification field. Extracting that channel and decrypting its AES-256-GCM
envelope yields an authenticated flag. Challenge crafted by **gopwnit**.

## Solution

### Step 1: Decode the ping timing

The attachment `1791619772841-dist.zip` contains `dist/ping.pcap`, with 4,400
packets. Filter for ICMP echo requests from `10.20.0.57` to `192.0.2.9`:
there are 1,300 requests. The other ping flow and UDP traffic are background noise.

Consecutive request timestamps have two separated interval ranges:

| Interval | Bit |
| --- | --- |
| 0.980008–1.019967 seconds | `0` |
| 1.230075–1.269947 seconds | `1` |

Use a 1.125-second threshold and pack bits most-significant-bit first. The decoded
stream begins with `aa 55`, followed by the one-byte message length `5c` (92), then:

```text
IP-ID low byte of each request: u16be L, nonce[12], AES-256-GCM. key=SHA256('TIDEMARK-0619')
```

### Step 2: Extract and authenticate the hidden message

Concatenate the low byte of the IP-ID field from the same requests in time order.
The first two bytes are `00 55`: a big-endian length of 85 bytes for the following
envelope. Split those 85 bytes into:

- **Nonce:** 12 bytes, `3baae682adacbba053084311`
- **Ciphertext:** 57 bytes
- **Authentication tag:** 16 bytes, `f16528e0a415df4e3f188b41255db8ae`

The key is the raw 32-byte SHA-256 digest of the ASCII string `TIDEMARK-0619`.
AES-GCM decryption with no additional authenticated data successfully verifies
the tag and recovers the flag.

The ninth payload byte across successive requests also spells
`GPI{1cmp_d4t4_f13ld_3xf1l_cl4ss1c_7e2b}`. This is a decoy; the timing instructions
lead to the authenticated result below.

### Step 3: Reproduce

Tools used: Python 3.11.2 standard library and PyCryptodome 3.24.0. The saved
[solve.py](solve.py) reads either the original ZIP or the extracted PCAP.
From the repository root:

```bash
python -m pip install pycryptodome
python Jitter/solve.py
```

Complete minimal solve, also runnable from the repository root:

```python
import hashlib
import re
import struct
import zipfile
from Crypto.Cipher import AES

with zipfile.ZipFile('Jitter/1791619772841-dist.zip') as archive:
    capture = archive.read('dist/ping.pcap')
assert capture[:4] == b'\xd4\xc3\xb2\xa1'
requests = []
offset = 24
while offset < len(capture):
    sec, usec, size, _ = struct.unpack_from('<IIII', capture, offset)
    frame = capture[offset + 16:offset + 16 + size]
    offset += 16 + size
    if frame[12:14] != b'\x08\x00':
        continue
    ip = frame[14:]
    ihl = (ip[0] & 15) * 4
    if (ip[9] == 1 and ip[12:16] == bytes([10, 20, 0, 57])
            and ip[16:20] == bytes([192, 0, 2, 9]) and ip[ihl] == 8):
        requests.append((sec * 1_000_000 + usec, ip[5]))
requests.sort()
bits = ''.join('1' if b[0] - a[0] > 1_125_000 else '0'
               for a, b in zip(requests, requests[1:]))
timing = bytes(int(bits[i:i + 8], 2) for i in range(0, len(bits) - 7, 8))
assert timing[:2] == b'\xaa\x55'
hint = timing[3:3 + timing[2]].decode('ascii')
phrase = re.search(r"SHA256\('([^']+)'\)", hint).group(1)
stream = bytes(low for _, low in requests)
length = int.from_bytes(stream[:2], 'big')
envelope = stream[2:2 + length]
key = hashlib.sha256(phrase.encode('ascii')).digest()
cipher = AES.new(key, AES.MODE_GCM, nonce=envelope[:12])
flag = cipher.decrypt_and_verify(envelope[12:-16], envelope[-16:])
print(flag.decode('ascii'))
```

Saved solver output, verified against both ZIP and PCAP inputs:

```text
Timing hint: IP-ID low byte of each request: u16be L, nonce[12], AES-256-GCM. key=SHA256('TIDEMARK-0619')
AES-GCM authentication: verified
GPI{th3_p4yl04d_w4s_n01s3_th3_g4ps_w3r3_th3_m3ss4g3_d5b0}
```

## Flag

```text
GPI{th3_p4yl04d_w4s_n01s3_th3_g4ps_w3r3_th3_m3ss4g3_d5b0}
```
