---
title: "double cross"
ctf: "GoPWNIt CTF 2026"
date: 2026-10-10
category: crypto
difficulty: easy
points: 50
flag_format: "GPI{...}"
author: "nomadrai"
---

# double cross

## Summary

The message is encrypted with two columnar transpositions. Semaphore and timed
Morse signals provide the keywords, and the existing solver reverses the
transpositions before passing the plaintext to the supplied answer checker.

## Solution — probable approach

The keywords and algorithm are preserved in [solve.py](solve.py); the final
checker output is not. The following documents that retained approach.

### 1. Read the two signaled keywords

The script records `key1.png`'s semaphore word as **HARBOUR**. Its transcription
of `key2.gif` uses 200 ms illuminated frames as dots and 600 ms illuminated
frames as dashes, with 600 ms dark letter gaps and a 1400 ms word-ending gap:

```text
.-.. / .- / -. / - / . / .-. / -. = LANTERN
```

These are saved intermediate keys, not newly decoded observations.

### 2. Undo both transpositions in reverse order

Read `dist/message.txt`. Decrypt with `LANTERN` first and `HARBOUR` second.
Sort column indices by `(keyword_letter, original_index)` so repeated letters
retain a stable left-to-right ordering.

For message length `L` and keyword width `w`, let `q,r=divmod(L,w)`.
Columns with original indices below `r` contain `q+1` characters; the others
contain `q`. Distribute ciphertext in sorted-key order, then read the restored
grid row by row, omitting nonexistent cells in its last row.

### 3. Check the reconstructed message

The saved solver re-encrypts the plaintext with both keywords and requires an
exact match to the original ciphertext. It then calls `dist/unlock.py` with
the recovered message. Neither operation was executed for this handoff.

## Tools and lesson

The local implementation uses Python's standard library and the supplied
checker. In double transposition, both reverse key order and uneven column
lengths matter; simply drawing a padded rectangle changes the message.

## Flag

**Submission note (participant-provided):** The flag was not saved at the moment
of submission due to hurry. The intermediate keywords `HARBOUR` and `LANTERN`
are retained in the solver.
