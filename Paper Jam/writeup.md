---
title: "Paper Jam"
ctf: "GoPWNIt"
date: 2026-10-10
category: misc
difficulty: easy
points: 50
flag_format: "GPI{...}"
author: "nomadrai"
---

# Paper Jam

Challenge author: **gopwnit**.

## Summary

The final harbour notice is a PDF incremental update: its original draft still exists in the file. Recovering that draft's invisible text through its actual font glyph encoding, rather than its misleading Unicode mapping, reveals **THE FIRST DRAFT**, which unlocks the flag.

## Solution

### 1. Recover the original content and read the glyphs

The archive contains `dist/notice.pdf` and `dist/unlock.py`. Inspection with `unzip -l`, `file`, and `strings -a` reveals two `%%EOF` markers and two versions of object `4 0`. The final trailer has `/Prev 1243`, linking to the earlier cross-reference table.

The first object 4 is encoded with `/ASCII85Decode` followed by `/FlateDecode`. Its decoded content includes the draft heading and text using font `/F2` with `3 Tr`, the invisible text-rendering mode. Each character has an explicit position, and the drawing commands are shuffled.

Font `/F2` points to object 7 for its actual character-to-glyph encoding:

```text
/Differences [1 /A /D /S /F /T /T /space /R /E /R /H /space /T /F /I]
```

Sort the characters by horizontal position and apply those glyph names:

```text
Character codes: 06 0B 09 0C 04 0F 0A 03 05 07 02 08 01 0E 0D
Glyph phrase:   THE FIRST DRAFT
```

### 2. Authenticate and decrypt the answer

The supplied checker normalizes the phrase to `thefirstdraft`, derives 128 bytes with scrypt, authenticates a ciphertext with HMAC-SHA256, and XOR-decrypts it. The following complete script runs from the directory containing the original ZIP and needs only Python 3's standard library:

```python
import ast
import base64
import hashlib
import hmac
import re
import zipfile
import zlib

with zipfile.ZipFile("1791632091371-dist.zip") as archive:
    pdf = archive.read("dist/notice.pdf")
    checker = archive.read("dist/unlock.py").decode()

objects = {}
for m in re.finditer(rb"(?m)^(\d+) 0 obj\s*\n(.*?)\nendobj", pdf, re.S):
    objects.setdefault(int(m[1]), m[2])  # Keep the original revision.

header, payload = objects[4].split(b"stream\n", 1)
length = int(re.search(rb"/Length\s+(\d+)", header)[1])
draft = zlib.decompress(base64.a85decode(payload[:length], adobe=True))
differences = re.search(rb"/Differences\s*\[([^]]+)\]", objects[7])[1]
tokens = differences.split()
glyphs = {
    code: " " if token == b"/space" else token[1:].decode()
    for code, token in enumerate(tokens[1:], start=int(tokens[0]))
}
positions = re.findall(rb"1 0 0 1 (\d+) (\d+) Tm <([0-9A-Fa-f]+)> Tj", draft)
positions.sort(key=lambda p: (-int(p[1]), int(p[0])))
phrase = "".join(glyphs[int(code, 16)] for x, y, code in positions)
print(phrase)

salt = bytes.fromhex(re.search(r'SALT = bytes.fromhex\("([0-9a-f]+)"\)', checker)[1])
boxes = ast.literal_eval(re.search(r"(?m)^BOXES = (.+)$", checker)[1])
key = re.sub(r"[^a-z0-9]", "", phrase.lower()).encode()
dk = hashlib.scrypt(key, salt=salt, n=2**17, r=8, p=4, maxmem=2**28, dklen=128)
for encoded in boxes:
    box = bytes.fromhex(encoded)
    tag, ciphertext = box[:32], box[32:]
    if hmac.compare_digest(hmac.new(dk[:32], ciphertext, "sha256").digest(), tag):
        print(bytes(a ^ b for a, b in zip(ciphertext, dk[32:])).decode().strip())
        break
else:
    raise ValueError("No authenticated flag")
```

The saved `solve.py` implements this path and also displays the misleading Unicode phrase. Reproduce with:

```bash
python3 solve.py
```

Observed output:

```text
Glyph phrase: THE FIRST DRAFT
Copy/paste mapping: COPY AND PASTE
GPI{1ncr3m3nt4l_upd4t3s_k33p_th3_0ld_p4g3_a9d3}
```

The original checker independently confirms the result:

```bash
python3 dist/unlock.py "THE FIRST DRAFT"
```

### Candidate provenance

There are deliberate competing flag paths in the attachment:

| Source | Result | Interpretation |
| --- | --- | --- |
| Actual glyph encoding of the original draft | `GPI{1ncr3m3nt4l_upd4t3s_k33p_th3_0ld_p4g3_a9d3}` | Selected solution; matches the PDF's retained revision and true glyph phrase. |
| `/ToUnicode` mapping, spelling `COPY AND PASTE` | `GPI{tr41l1ng_d4t4_t3lls_t4l3s_0p3n_s0urc3_1nt3l_w1ns_c7b6}` | Misleading extraction path; the checker also authenticates this phrase, so authentication alone does not distinguish the intended answer. |
| Base64 `ref` in the final page | `GPI{th3_l0gs_r3m3mb3r_c4ch3d_c0p13s_l1v3_0n_0c8e}` | Flag-shaped reference unrelated to recovering the original glyph phrase. |

Validation was local against the supplied checker; no platform submission was performed. The key lesson is that incremental PDF updates retain older objects, and `/ToUnicode` text extraction can disagree with the glyphs a font actually draws.

## Flag

```text
GPI{1ncr3m3nt4l_upd4t3s_k33p_th3_0ld_p4g3_a9d3}
```
