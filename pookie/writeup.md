---
title: "pookie"
ctf: "GoPWNIt CTF 2026"
date: 2026-10-10
category: web
difficulty: hard
points: 150
flag_format: "GPI{...}"
author: "nomadrai"
---

# pookie

## Summary

AegisVault Web-Enclave v2.0 advertises a serverless, zero-trust asset enclave.
The most probable approach is to trace where its client receives encrypted
assets, key material, and authorization state, then identify whether the browser
is trusted to enforce a restriction it can also bypass.

## Solution — probable approach

The dashboard identifies `https://web-gilt-six-47.vercel.app/` as the target.
No source bundle, request transcript, or recovered key is saved locally.
Client-side trust failure is an investigation hypothesis suggested by the
product description, not a verified implementation detail.

### 1. Trace asset delivery and unsealing

Inspect the frontend's scripts and normal network exchanges. Follow the
functions responsible for loading an asset, checking access, acquiring a
decryption key, and displaying plaintext. Determine which checks are performed
by a backend and which exist only in browser code or browser storage.

### 2. Identify a client-controlled trust decision

The probable weakness is a UI-only access gate protecting material already
delivered to the browser, or an asset-unsealing function that accepts
client-controlled privilege state. Follow the actual data dependencies: a
hardcoded key, reversible key derivation, or exposed unsealing input would
need to be demonstrated before claiming that the encrypted asset can be opened.

The slogan alone does not establish a particular cipher, token format, or
WebAssembly component, so this account does not invent one.

### 3. Reproduce the unsealing path

Use the recovered inputs and the application's own transformation to open
the protected asset. Verify any supplied authentication tag and preserve the
exact resulting `GPI{...}` value rather than trusting a changed UI label.

## Tools and lesson

Probable tools: browser developer tools, source inspection, and an offline
decoder matching the discovered format. A browser cannot keep a decryption
secret from its user when both the secret and protected material are delivered
to that same client.

## Flag / key

**Submission note (participant-provided):** The flag/key was not saved at the
moment of submission due to hurry.
