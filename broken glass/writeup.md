---
title: "broken glass"
ctf: "GoPWNIt CTF 2026"
date: 2026-10-10
category: forensics
difficulty: easy
points: 50
flag_format: "GPI{...}"
author: "nomadrai"
---

# broken glass

## Confirmed result

```text
GPI{f1x_th3_h34d3r_f1x_th3_byt3_r34d_th3_p1x31s_71d8}
```

**Submitted and accepted: +50 points, #7 Solve.**
The accepted text contains `p1x31s`: the character after `3` is the **digit `1`**.
The initial image transcription used `p1x3ls` with a lowercase `l` and was rejected.

The normal submission API returned HTTP 200 on 2026-10-10 at 17:39:50 UTC:

```json
{
  "data": {
    "message": "Flag accepted.",
    "rank": "#7 Solve",
    "points": 50
  }
}
```

An independent query of the account's solved challenges confirmed that
`broken-glass-520176` was present. The full submission history is in
[submission-results.json](submission-results.json).

## Summary

The attachment contains a ZIP with a damaged central directory and a corrupted
PNG. Recover the files from their local ZIP headers, repair the PNG using its
surviving CRCs, and read the visible text. Challenge crafted by gopwnit.

## Solution

### 1. Recover the ZIP entries

The outer `1791637322746-dist.zip` opens normally and contains
`dist/evidence.zip` (157,175 bytes). The inner archive lacks an
end-of-central-directory record. A damaged directory fragment starts at `0x265dc`.

Scan for the local-header signature `PK\x03\x04`:

| Entry | Header offset | Payload offset | Recovered bytes |
|---|---:|---:|---:|
| `evidence/README.txt` | `0x0` | `0x31` | 68 |
| `evidence/thumb.png` | `0x75` | `0xa5` | 75,962 |
| `evidence/photo.png` | `0x12978` | `0x129a8` | 80,960 |

The README is stored without compression. Both PNG entries use DEFLATE, have
ZIP flag bit 3 set, and contain zero sizes in their local headers. Decode each
raw DEFLATE stream with `zlib.decompressobj(-15)`. Its end marker identifies the
payload boundary without needing the central directory.

### 2. Repair the photo

The PNG signature and IHDR width/height fields were zeroed. Its chunk layout and
original checksums survive. Two IDAT chunks fail their CRC checks. Trying
single-byte substitutions identifies the corrections matching the original CRCs:

| IDAT index (zero-based) | Chunk offset | Payload byte index | Correction |
|---:|---:|---:|---|
| 3 | `0x1801` | 387 | `0xc1` → `0x3d` |
| 17 | `0x8609` | 672 | `0x4e` → `0xa1` |

The corrected IDAT stream decompresses to **94,146 bytes** and passes its zlib
Adler-32 checksum. The remaining IHDR fields specify 8-bit grayscale without
interlacing, so each row has one filter byte and `width` pixel bytes:

```text
(width + 1) × height = 94146
```

Enumerate factor pairs and compare each reconstructed IHDR to the original CRC
`0x1883c02e`. The unique match is **1206 × 78**. Restore those dimensions and the
standard PNG signature. Every original PNG chunk CRC now passes.

### 3. Read and validate the text

Run the recovery script from the project root:

```bash
python "broken glass/solve.py"
```

The script uses Python 3's standard library and starts with the original outer
ZIP. It writes [recovered/photo-repaired.png](recovered/photo-repaired.png).
Its final printed flag records the subsequently platform-confirmed spelling;
it does not perform OCR or claim that checksums validate that spelling.

The image was initially transcribed with `p1x3ls`. After the server explicitly
rejected that candidate, testing the digit-`1` variant `p1x31s` succeeded.
This exact distinction was the final blocker. The README, thumbnail, and PNG
metadata also contain flag-shaped strings; all three were explicitly rejected.

## Additional forensic verification

To rule out another hidden layer, the image generator was reconstructed using
NumPy's `default_rng(0xB20C)`:

1. Generate `normal(205, 22, (78, 1206))`, clip to `[0,255]`, and cast to `uint8`.
   This reproduces **86,411 background pixels** exactly.
2. Continue with `normal(40, 15, 7657)` to reproduce all **7,657 text pixels**.
3. Two pairs of `integers(50,1950)` and `integers(1,256)` yield exactly the
   corruption positions and XOR masks: `(387,252)` and `(672,239)`.
4. Continue the same background/text process for the thumbnail: all **85,488
   pixels** match, including its **6,938 text pixels**.
5. The next `rng.bytes(23)` exactly reproduces the unexplained ZIP trailer.

Thus the image noise and trailer are deterministic generated data. All repeated
characters in the photo also have identical glyph masks. The compressed photo
IDAT and all ZIP DEFLATE streams match ordinary zlib recompression byte for byte.

This verification is saved in [verify_noise.py](verify_noise.py), requiring
NumPy and Pillow. It writes a noise-free image to
[recovered/photo-noise-removed.png](recovered/photo-noise-removed.png).

## Live attachment integrity

An authenticated download confirmed that the live attachment is byte-for-byte
identical to the original local file. SHA-256:

```text
2ea6a9b929b66b2c0fc78828f8f2fc1902868d151bae5d235a81314d32f62043
```

Initial API attempts encountered a 20-request/900-second rate limit. Those HTTP
429 responses did not establish flag correctness. Subsequent HTTP 400 rejections
and the final HTTP 200 acceptance were recorded separately.
