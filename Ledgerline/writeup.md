---
title: "Ledgerline"
ctf: "GoPWNIt CTF 2026"
date: 2026-10-10
category: web
difficulty: hard
points: 150
flag_format: "CTF{...}"
author: "nomadrai"
---

# Ledgerline

## Summary

The challenge asks the player to prove a claim and make the server believe it.
The most probable route is a flaw in how a proof or signed assertion is bound
to the identity, resource, and action it is supposed to authorize.

## Solution — probable approach

The only retained basis is the dashboard description and target
`http://15.252.240.95`. No proof format, verification code, or request transcript
is saved; the following is a bounded reconstruction rather than a claim about
a specific cryptographic protocol.

### 1. Identify the statement the server verifies

Follow the normal account and ledger workflow with developer tools. Identify
the request that carries a proof, signature, token, or assertion and the
protected action that consumes it. Record which values describe the caller,
ledger object, requested action, freshness, and challenge issued by the server.

### 2. Check that the proof covers the authorization decision

The likely weakness is incomplete binding: the verifier accepts evidence for
one statement while a separate client-controlled field selects the privileged
identity or ledger operation. Compare the bytes or fields actually verified
with the fields used afterward to authorize access.

A probable exploit would preserve a valid proof while changing an unbound
authorization field. Demonstrating that distinction is essential; merely
editing a displayed identity or an unverified token is insufficient. No
specific algorithm or forged proof is asserted without a retained record.

### 3. Access the protected ledger result

Use the accepted assertion in the relevant protected operation and extract
the returned `CTF{...}` value. The description excludes attacking the CAPTCHA;
the intended reasoning concerns the application's own verification boundary.

## Tools and lesson

Probable tools: an HTTP proxy, browser developer tools, and a parser for the
actual assertion format. Cryptographic validity and authorization are separate:
a valid proof must cover the exact identity, resource, and action being trusted.

## Flag

**Submission note (participant-provided):** The flag was not saved at the moment
of submission due to hurry.
