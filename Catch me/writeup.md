---
title: "Catch me"
ctf: "GoPWNIt CTF 2026"
date: 2026-10-10
category: misc
difficulty: medium
points: 100
flag_format: "GPI{...}"
author: "nomadrai"
---

# Catch me

## Summary

The challenge presents fifty tactical buttons: forty-eight return deceptive
Base64 messages, while exactly two supply genuine pieces for a flag recombiner.
The probable solution is to distinguish those pieces using the application's
selection and recombination logic.

## Solution — probable approach

This account is reconstructed from the dashboard's explicit rules and target,
`https://osint-seven-flax.vercel.app/`. Button identifiers, recovered halves,
and a browser transcript are not retained locally.

### 1. Inspect how button results are selected

Read the page's event handlers and associated JavaScript to identify the mapping
between the fifty controls and their results. Follow the code used by the
Flag Recombiner to understand what distinguishes a genuine fragment from a
display-only decoy. Base64 is only an encoding; readable output by itself does
not authenticate a candidate.

### 2. Isolate the two genuine halves

Associate each result with its button, decode the transport encoding, and
compare it with the fragment format expected by the recombiner. The most
probable discriminator is the application's own validation or construction
logic, rather than whether a decoded message looks convincing.

Preserve exact byte content, padding, and fragment order. Neither the two
button numbers nor a particular cipher can be established from the saved
description, so this writeup does not supply guessed values.

### 3. Recombine and verify the result

Provide the selected pair to the original Flag Recombiner in its expected
order. Follow its complete transformation and require its genuine success
result before taking the resulting `GPI{...}` string as the answer.

## Tools and lesson

Probable tools: browser source inspection, developer tools, and a Base64 decoder.
When a challenge explicitly supplies many decoys, validate candidates against
the mechanism that constructs the answer instead of trusting surface appearance.

## Flag

**Submission note (participant-provided):** The flag was not saved at the moment
of submission due to hurry.
