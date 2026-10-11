---
title: "Quietroom"
ctf: "GoPWNIt CTF 2026"
date: 2026-10-10
category: crypto
difficulty: medium
points: 100
flag_format: "GPI{...}"
author: "nomadrai"
---

# Quietroom

## Summary

Repeated noisy timing captures reveal the square-and-multiply sequence of one
secret exponent. The saved solver classifies a reliable prefix and uses the
published group element to check a small, constrained suffix search.

## Solution — probable approach

The reconstruction follows [solve.py](solve.py) and the retained `handout/`
files. No exponent search or decryption was executed for this writeup.

### 1. Classify the stable timing prefix

Load `params.txt` and the 256 runs in `traces.csv`. The existing implementation
discards timings at or above 2.0 microseconds to remove the contention
population. It averages corresponding operation positions only while at least
100 clean measurements remain.

Because the leading exponent bit is one, the first two operations are a square
and a multiply. Use their timing means to initialize two clusters, then label
the reliable prefix as `S` or `M`. The resulting sequence must start with `SM`
and contain no consecutive multiplies.

### 2. Decode exponent bits and constrain the suffix

An `S` followed by `M` encodes one; an `S` followed by another `S` encodes zero.
Leave a final unmatched square unresolved. The saved code uses a total of 238
operations and a 160-bit exponent, implying 78 set bits. This fixes how many
ones must occur in the remaining suffix.

Enumerate suffixes with precisely that Hamming weight and accept a candidate
only when `pow(g, x, p) == y`. The public value is a verifier for the recovered
exponent, not a reason to attempt a generic discrete logarithm.

### 3. Derive the flag mask

The solver hashes the exponent's fixed-width, big-endian encoding with SHA-256
and repeats that digest as an XOR mask over the hex-decoded `flag.enc`. Preserve
the declared exponent width when serializing the key material.

## Tools and lesson

The retained implementation uses Python's standard library. Aggregate noisy
measurements first, and use mathematical constraints to handle the least
reliable tail rather than forcing every timing sample into a bit decision.

## Flag

**Submission note (participant-provided):** The flag was not saved at the moment
of submission due to hurry.
