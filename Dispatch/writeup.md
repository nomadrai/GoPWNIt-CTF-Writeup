---
title: "Dispatch"
ctf: "GoPWNIt CTF 2026"
date: 2026-10-10
category: web
difficulty: medium
points: 100
flag_format: "CTF{...}"
author: "nomadrai"
---

# Dispatch

## Summary

The dashboard asks whether the player can see what happens inside a dispatcher.
The most probable route is to inspect its job-submission interface for a
server-side request or routing operation that exposes an internal-only resource.

## Solution — probable approach

The saved basis is the description and competition target
`http://52.66.248.135`. No local application source, request history, or final
payload is retained. The server-side request hypothesis below is inferred from
the description, not an established vulnerability.

### 1. Trace a normal dispatch request

Observe the browser's requests while using the dispatch workflow and inspect
the frontend code that constructs them. Determine whether jobs contain a
destination, callback, resource identifier, or another routing value that the
backend resolves. Identify the response or status page that exposes job output.

### 2. Follow the routing trust boundary

The probable issue is user-controlled routing reaching a resource intended
only for the dispatcher. A complete investigation would establish that the
request is actually made by the server and that internal content is returned
through the job result, rather than merely displaying a supplied URL.

If a destination is filtered, compare the validation step with the eventual
resolution step to identify the relevant mismatch. No particular internal
hostname, port, path, or parser bypass is preserved, so none is invented here.

### 3. Recover the protected result

Use the exposed internal dispatch output to locate the clearance or flag value
and preserve the exact `CTF{...}` string. The dashboard explicitly excludes
attacking the CAPTCHA; it is not part of this probable solution path.

## Tools and lesson

Probable tools: browser developer tools and an HTTP proxy. A dispatcher must
validate the resource it ultimately accesses, not just the shape of a
client-provided routing field.

## Flag

**Submission note (participant-provided):** The flag was not saved at the moment
of submission due to hurry.
