---
title: "college_crush"
ctf: "GoPWNIt CTF 2026"
date: 2026-10-10
category: reverse
difficulty: medium
points: 100
flag_format: "GPI{...}"
author: "nomadrai"
---

# college_crush

## Summary

The challenge description identifies a custom username/password crackme, and
the local attachment contains `college_crush.exe`. The probable solution is
to reconstruct the credential-validation routine and recover the input that
reaches its genuine success branch.

## Solution — probable approach

Only the archive and extracted executable are retained. This is a proposed
analysis path, not a report of newly performed disassembly or debugging.

### 1. Trace input handling to the validation routine

Extract `1791624486615-college_crush.7z` and identify the executable's architecture
and runtime before choosing a disassembler. Locate references to its login
prompts, rejection messages, and success messages. Follow the input buffers
through length checks, normalization, and the final comparison.

The challenge description is about two credentials, so determine whether the
password is independently checked or derived from the username. Preserve the
program's byte encoding and case sensitivity when modeling those inputs.

### 2. Reconstruct the accepted credential relationship

Translate the validation expressions and embedded comparison data into a small
offline model. Work backward from the expected values, resolving transformations
such as character arithmetic or table lookups only where they are actually
present in the binary. A debugger at the comparison boundary would reveal
whether the model matches the program's own intermediate values.

### 3. Confirm the success output

Validate the recovered username/password against the unmodified checker and
read the success output for the intended submission value. The advertised
wrapper is `GPI{...}`; no preserved record establishes the exact credentials
or whether the flag body is a password or a separate success string.

## Tools and lesson

Probable tools: a 7z extractor, Ghidra or another PE disassembler, and a Windows
debugger. Trace the real acceptance logic rather than treating arbitrary
printable strings or a patched success branch as a recovered key.

## Flag / key

**Submission note (participant-provided):** The flag/key was not saved at the
moment of submission due to hurry.
