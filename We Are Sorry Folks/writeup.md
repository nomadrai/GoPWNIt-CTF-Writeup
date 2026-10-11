---
title: "We Are Sorry Folks"
ctf: "GoPWNIt CTF 2026"
date: 2026-10-10
category: web
difficulty: hard
points: 150
flag_format: "GPI{...}"
author: "nomadrai"
---

# We Are Sorry Folks

## Summary

The challenge, listed under **Other**, links to a form whose confirmation sends
players to `https://hell0-u3er.vercel.app/`. That page presents a maintenance
message and redirects attention toward the leaderboard, but its hidden footer
click handler reveals the flag locally.

## Solution

### 1. Decode the page source

The response consists of `document.write(decodeURIComponent(escape(atob(...))))`.
Base64-decoding the argument reveals the HTML and its inline script. The text
"Wrong destination Folks!! Try searching on the leaderboard" is a decoy.

### 2. Reproduce the hidden interaction

The year **2026** in the footer has a click listener. Fifteen clicks reveal the
flag; a gap of two seconds resets the count. This matches the description's
references to repeated knocks and time.

The handler concatenates three fragments, in this order:

| Location | Value |
|---|---|
| Year span `data-z` | `NWcoUAUvTxVla3hSI2Fq` |
| CSS custom property `--a` | `Ln5Pdw8JEB9MPSZISWYF` |
| Body `data-k` | `PREFTD86FSNqMj9aNUMd` |

It base64-decodes the combined string, reverses the resulting bytes, and XORs
them with the repeating key `[90, 19, 124, 33, 72]`.

Complete standalone decoder:

```python
import base64

encoded = (
    "NWcoUAUvTxVla3hSI2Fq"
    "Ln5Pdw8JEB9MPSZISWYF"
    "PREFTD86FSNqMj9aNUMd"
)
key = [90, 19, 124, 33, 72]
data = base64.b64decode(encoded)[::-1]
print(bytes(v ^ key[i % len(key)] for i, v in enumerate(data)).decode())
```

The saved [solve.py](solve.py) extracts the same values directly from
[page-decoded.html](page-decoded.html), using only Python's standard library.

## Flag

```text
GPI{why_4re_y0u_u5ing_c1AUd3_f0r_s01ving_CTF}
```

## Verification

Submitted to challenge `we-are-sorry-folks-ce8b6e` using the authorized account.
At **2026-10-10 18:21:08 UTC**, the API returned HTTP 200:

```json
{"message":"Flag accepted.","rank":"#26 Solve","points":150}
```

A separate solved-list request confirmed the challenge was solved.
Full confirmation: [submission-result.json](submission-result.json).
