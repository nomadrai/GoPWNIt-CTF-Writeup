---
title: "Hips_Donot_Loop"
ctf: "GoPWNIt CTF 2026"
date: 2026-10-10
category: reverse
difficulty: hard
points: 150
flag_format: "GPI{...}"
author: "nomadrai"
---

# Hips_Donot_Loop

## Summary

The choreography program combines structured motion records, a stateful cadence
check, and a Feistel transform into three flag fragments. The retained
[solve.py](solve.py) documents the reconstruction, but its final output was not
saved locally.

## Solution — probable approach

This reconstruction is based on the existing solver's logic, not a new execution.

### 1. Recover the choreography fragment

The script parses `rehearsal.choreo` as a `CHOR` header followed by 24
sixteen-byte records. Select the eight records with tag `0x4849` and an active
flag, then sort them by beat. Extract their cue bytes and apply the executable's
eight-byte permutation, a five-bit byte rotation, and XOR mask to obtain the
first fragment.

### 2. Reconstruct the cadence and reverse the final transform

The saved code initializes a sixteen-bit state at `0x5a17`, incorporates the
first fragment, and compares eight possible moves against each checkpoint.
Each selected move chooses a character from one row of an eight-by-eight grid.
Those characters form the second fragment.

Hash `part1 + b"_" + part2` with SHA-256 and interpret the first 24 digest bytes
as six little-endian round keys. Reverse the six-round Feistel network stored
in the binary to obtain the final eight-byte fragment. Preserve sixteen- and
thirty-two-bit wraparound exactly as the saved implementation does.

### 3. Apply the original verification criteria

The solver assembles three underscore-separated fragments inside `GPI{...}`
and compares their SHA-256 digest with the executable's embedded digest. Its
optional native verification feeds the move sequence and reconstructed string
to `encore`, requiring the `Scoreboard Flag Verified` response.

## Tools and lesson

The existing solver uses Python's standard library; native verification targets
the supplied ELF. Structured input records and small state transitions reduce
the task to deterministic reconstruction rather than guessing complete flags.

## Flag

**Submission note (participant-provided):** The flag was not saved at the moment
of submission due to hurry.
