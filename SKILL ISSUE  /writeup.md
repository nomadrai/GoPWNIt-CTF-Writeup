---
title: "SKILL ISSUE"
ctf: "GoPWNIt CTF 2026"
date: 2026-10-10
category: misc
difficulty: medium
points: 100
flag_format: "SKILL{...}"
author: "nomadrai"
---

# SKILL ISSUE

## Summary

The challenge provides a fictional incident archive containing a canary commit,
an untrusted staging build, and an authoritative production release. Select the
release using the audit and advisory, verify the production file checksums, and
apply the postmortem's PBKDF2 recipe to derive the flag.

## Solution

### 1. Select the authoritative evidence

Download the evidence bundle from:

```text
https://skill-issue-ctf-blue.vercel.app/api/evidence/download
```

In `gitforge_audit.json`, advisory `SEC-ADV-2026-004` revokes `0xDEADBEEF99`
and requires maintainer key `0x4A89C2F150` for subsequent commits. Commit
`7b32a0d84f` belongs to the canary daemon and has a revoked/expired signature.

The production containment release is **`c48e89f2a6`**, with signer UID
**`xero_entropy@heuristi.core`**, a `VALID` audit status, and a message specifying
the rebase onto `origin/hardened-core`. The baseline commit is valid but is not
the containment release. These signature statuses are supplied audit assertions;
the bundle does not contain a public keyring or detached signatures for GPG
verification.

Telemetry places canary deployment on April 11 and the production release on
April 14. Chrony records a `-840.124219`-second correction on April 12. Production
commit verification, build verification, and failsafe engagement all occur after
that synchronization, at 19:04:22, 19:08:16, and 19:10:00 UTC on April 14.

### 2. Verify the production guard and derive the flag

Recompute SHA-256 over the exact production files. Both match `manifest.json`:

```text
dist/auth_guard.min.js:
f1d0505324d0168b7dab6a6d9d7a3fbaca448db84d3558fe15e00592b922c719

dist/core_engine.wasm:
a73fa9b03a68e5dad1ef3ac1cc19d47084d17884ac91967f326c7de4becbecd0
```

The production guard contains `sessionFailsafe:'REASONING_OVER_PROMPTS'`;
the telemetry independently records that exact locked status. The staging build
has a different token and is explicitly untrusted.

Following `postmortem_draft.md`, use hexadecimal strings encoded as UTF-8—not
raw digest bytes—for both the PBKDF2 password and salt:

```text
KEY = SHA256("xero_entropy@heuristi.core:c48e89f2a6").hexdigest()
    = 6aacf8b00f6a525f8c1f88981f72db64b3843a99864bef15ddb014f546cec812

SALT = SHA256(production_guard).hexdigest()[:16]
     = f1d0505324d0168b

DERIVED = PBKDF2-HMAC-SHA256(KEY.encode(), SALT.encode(), 10000, dklen=16)
        = faf6e03adfb6964dd375b9cca79b4ab6
```

The adjacent `solve.py` reads the original ZIP, selects the release, checks both
production checksums, corroborates the status token, and prints the complete flag:

```bash
python "SKILL ISSUE/solve.py"
```

### 3. Verify

The challenge's `/api/submit-flag` endpoint returned
`success: true, verdict: AUTHENTIC_SOLVE`.

The GoPWNIt scoreboard accepted the same flag on **2026-10-10 at 17:38:37 UTC**:

```json
{"message": "Flag accepted.", "rank": "#20 Solve", "points": 100}
```

## Flag

```text
SKILL{faf6e03adfb6964dd375b9cca79b4ab6_REASONING_OVER_PROMPTS}
```
