---
title: "Sprintly"
ctf: "GoPWNIt CTF 2026"
date: 2026-10-10
category: web
difficulty: medium
points: 100
flag_format: "CTF{...}"
author: "nomadrai"
---

# Sprintly

## Summary

The clues ask what exists beyond REST and mention code executing from unusual
locations. The most probable path is a secondary API, plausibly GraphQL, whose
resolver or job-processing feature evaluates attacker-influenced input.

## Solution — probable approach

The retained source is the dashboard description for `http://43.204.115.83`.
There is no saved schema, payload, or execution transcript. GraphQL and an
unsafe execution sink are hypotheses suggested by the wording.

### 1. Identify the non-REST interface

Inspect the frontend's network traffic and JavaScript for query documents,
operation names, and its actual API URL. Use the discovered interface to learn
the operations involved in tasks, automation, or other server-side processing.
If the interface exposes its schema, use that to understand input types and
available mutations rather than guessing REST routes.

### 2. Trace input to the execution feature

Follow the operation that accepts structured expressions, templates, or job
configuration into the server-side processing path. The likely flaw is an
input field interpreted as executable content instead of inert data. Establish
the execution context with a harmless deterministic expression before deriving
an exploit for that specific runtime.

The description does not identify a language, resolver name, library, or CVE.
A writeup cannot responsibly turn those unknowns into an exact payload.

### 3. Read the challenge result in that context

Once the execution primitive is established, use the exposed application
context to retrieve the protected flag and preserve the server's returned
`CTF{...}` value. The CAPTCHA is explicitly outside the intended attack path.

## Tools and lesson

Probable tools: browser developer tools, an HTTP proxy, and an API client.
An application can expose a powerful secondary API even when its visible
interface appears to be ordinary REST-driven task management.

## Flag

**Submission note (participant-provided):** The flag was not saved at the moment
of submission due to hurry.
