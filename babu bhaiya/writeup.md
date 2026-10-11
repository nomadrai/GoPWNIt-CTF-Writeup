---
title: "babu bhaiya"
ctf: "GoPWNIt CTF 2026"
date: 2026-10-10
category: web
difficulty: medium
points: 100
flag_format: "GPI{...}"
author: "nomadrai"
---

# babu bhaiya

## Summary

The fictional Tatkal allotment portal presents an impossibly short booking
window, a payment-like prerequisite, and exhausted seats. The most probable
approach is to inspect its booking state machine and identify prerequisites
that the interface enforces more strictly than the server.

## Solution — probable approach

The retained basis is the dashboard description and target URL,
`https://babubhaiya-eight.vercel.app/`. No local request transcript or exploit
script is present. The workflow below is an inference, not a verified account
of specific endpoint names or payloads.

### 1. Map the portal's booking workflow

Use browser developer tools or an HTTP proxy to observe the requests behind
registration, eligibility, payment approval, and seat allotment. Inspect the
frontend handlers for the advertised opening time and the Window 4 prerequisite.
Record which fields represent server-issued state and which originate in
client-controlled form data or browser storage.

### 2. Test the trust boundary at the allotment step

The likely weakness is an allotment request that trusts a client-supplied
eligibility, approval, or workflow marker instead of deriving it from the
authenticated session. Reconstruct the normal request format, then determine
whether the final action can be reached without the interface's artificial
timing and prerequisite gates.

The narrative alone does not prove a race condition: a tiny displayed time
window can simply distract from missing server-side validation. The intended
path should be justified by the actual booking request and its response.

### 3. Read the server's completion result

After reaching the authorized-looking allotment state, inspect the completion
response or resulting allocation record for the flag. A changed display or
client-side success message alone would not establish completion.

## Tools and lesson

Probable tools: browser developer tools and an HTTP interception proxy.
Business prerequisites must be enforced at the state-changing server operation,
not only by disabled buttons, countdowns, or hidden client state.

## Flag

**Submission note (participant-provided):** The flag was not saved at the moment
of submission due to hurry.
