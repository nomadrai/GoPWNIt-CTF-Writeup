---
title: "Lost and found"
ctf: "GoPWNIt"
date: 2026-10-10
category: misc
difficulty: easy
points: 50
flag_format: "GPI{...}"
author: "nomadrai"
---

# Lost and found

## Summary

A tidied Git repository retains an unindexed, truncated temporary pack. Salvaging
its complete objects reveals 24 unlogged commits whose `part` files form a gzip
stream. Decompressing it recovers the phrase used by the supplied flag checker.

## Solution

### 1. Inspect the repository beyond its visible history

The attachment `1791629559718-dist.zip` contains `dist/harbour.git.tar` and
`dist/unlock.py`. The tar holds a bare repository with a normal indexed pack and
an additional file:

```text
harbour.git/objects/pack/tmp_pack_568fed
```

There are two earlier flag-like results to distinguish from the final recovery:

- Commit `b507907` contains a Base64-encoded `secret.txt` that decodes to
  `GPI{0ld_3nc0d1ngs_d13_h4rd_l00k_4t_th3_l4st_byt3s_74f9}`.
- The reflog exposes a 12-commit scratch chain ending at `2076cd5`. Concatenating
  its historical `part` files in parent order and decompressing with gzip gives
  `phrase: CHECK THE REFLOG`. The checker accepts this phrase but returns
  `GPI{th3_l0gs_r3m3mb3r_bl1nd_sp0ts_3v3rywh3r3_f3fa}`.

The remaining temporary pack is the key evidence: ordinary `git log` and the
original `git fsck` do not include its objects. The checker contains two separate
authenticated output boxes, so acceptance of the reflog phrase alone does not
establish that it is the intended final answer.

### 2. Salvage the temporary pack and reconstruct its hidden chain

On an extracted working copy, run:

```bash
git --git-dir=harbour.git unpack-objects -r \
  < harbour.git/objects/pack/tmp_pack_568fed
```

Git reports `fatal: early EOF`, but it has already written 70 recoverable objects.
These include 24 additional commits, absent from the supplied reflog. Their tip is:

```text
6c677db17bb84b330eeb56eb2191bb46c810fcd4
```

The chain branches from `87c3d0c5388dc155ece82df7fcc6306a3213012c`. Concatenating
each commit's `part` blob in oldest-to-newest order produces a 50-byte gzip stream:

```text
1f8b080000000000020a2bc8284a2c4eb552f0f30ff1f0f47357f00c56700d730d527071f5710d7175e102006a56a06720000000
```

Its decompressed contents are:

```text
phrase: NOTHING IS EVER DELETED
```

### 3. Verify with the supplied checker

```bash
python3 dist/unlock.py "NOTHING IS EVER DELETED"
```

Output:

```text
GPI{d4ngl1ng_c0mm1ts_n3v3r_r34lly_l34v3_7a3e}
```

The checker normalizes the phrase, derives a key with scrypt, verifies an
HMAC-SHA256 tag, and decrypts the corresponding output. This flag comes from the
deeper, unlogged recovery path. It has been verified locally with the supplied
checker; platform submission was not performed.

## Reproduction

The complete adjacent script, [`solve.py`](solve.py), starts from the original ZIP,
extracts a fresh temporary copy, identifies newly recovered commits automatically,
reassembles their fragments, and runs the original checker. It requires Python 3
with scrypt support and Git. It was verified with Python 3.11.2.

From the workspace root:

```bash
python3 "Lost and found/solve.py"
```

Expected output:

```text
Recovered 24 hidden commits; tip: 6c677db17bb84b330eeb56eb2191bb46c810fcd4
Phrase: NOTHING IS EVER DELETED
GPI{d4ngl1ng_c0mm1ts_n3v3r_r34lly_l34v3_7a3e}
```

## Flag

```text
GPI{d4ngl1ng_c0mm1ts_n3v3r_r34lly_l34v3_7a3e}
```

## Lesson

Removing branch references or reflog entries does not erase all repository data.
Even a truncated, unindexed pack can retain complete recoverable Git objects.
