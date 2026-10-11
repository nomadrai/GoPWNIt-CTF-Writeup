---
title: "SHORTCUTS"
ctf: "GoPWNIt CTF 2026"
date: 2026-10-10
category: reverse
difficulty: medium
points: 100
flag_format: "GPI{...}"
author: "nomadrai"
---

# SHORTCUTS

## Summary

`treasurebox` validates a sixty-four-byte key by transforming an eight-by-eight
grid. The saved [solve.py](solve.py) reverses those transformations and checks
branch consistency against the original program.

## Solution — probable approach

The following describes the retained solver; the recovered key and its output
were not retained, and the solver was not rerun for this handoff.

### 1. Extract the target and reconstruct the forward transform

The script reads eight rows of eight target bytes starting at file offset
`0x3020`, with a nine-byte stride between rows. Its forward model swaps adjacent
rows, adjacent columns, and diagonal pairs within two-by-two cells. A left-to-right
pass conditionally swaps neighboring characters according to the parity of
their sum. Two additional parity passes depend on intermediate grid values.
Finally, every byte is increased by two modulo 256.

### 2. Invert the operations and resolve conditional branches

Subtract two from each target byte. Enumerate the four combinations of the two
optional passes, undoing enabled parity passes from right to left. Reverse the
remaining operations in order; the fixed swaps are their own inverses.

Run each resulting candidate through the saved forward model. Retain only a
candidate that reproduces the entire target and takes the same conditional
branches assumed during inversion. This avoids treating branch-dependent
permutations as a single unconditional shuffle.

### 3. Validate with the original executable

The existing script supplies a retained candidate as standard input to
`treasurebox` and requires `The box happily accepts your key!`. It then wraps
the raw key in the challenge's `GPI{...}` format. That is the documented final
validation criterion, rather than a fabricated output transcript.

## Tools and lesson

The local artifacts include the RAR attachment, extracted `treasurebox`, and
a standard-library Python solver. Reverse iteration is essential when undoing
overlapping adjacent swaps.

## Flag / key

**Submission note (participant-provided):** The flag/key was not saved at the
moment of submission due to hurry.
