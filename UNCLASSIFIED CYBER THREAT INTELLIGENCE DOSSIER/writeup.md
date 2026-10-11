---
title: "UNCLASSIFIED CYBER THREAT INTELLIGENCE DOSSIER"
ctf: "Not specified in the challenge prompt"
date: 2026-10-10
category: osint
difficulty: hard
points: 150
flag_format: "FLAG{real_name_relocation_city_subkey_id}"
author: "nomadrai"
challenge_author: "bhaiya_ji"
---

# UNCLASSIFIED CYBER THREAT INTELLIGENCE DOSSIER

## Summary

The challenge mirror hosts a simulated OSINT investigation with searchable intelligence records. Its frontend and search API disclose the three attribution components, and its verification API accepted the reconstructed flag.

## Solution

### Step 1: Inspect the frontend and query its intelligence records

The working mirror was `https://outdoor-interface-briefing-tests.trycloudflare.com`.
Its HTML loads `/assets/index-Dul1-pOw.js`, which exposes `/api/search?q=...` and `/api/submit-flag`. The frontend specifies lowercase components, underscore separators, and a six-character subkey ID.

Querying `/api/search?q=sternenstaub_91` returns the record `snap_threat_intel_leviathan`, which provides all three components:

| Component | Evidence | Flag value |
|---|---|---|
| Real name | Bulletin: “Subject's authentic identity is Kaelen Voss, formerly of NTNU Trondheim.” | `kaelen_voss` |
| Relocation city | Bulletin identifies Tromsø; metadata gives `verified_city: Tromso`. | `tromso` |
| Genuine subkey | Bulletin: “YubiKey hardware subkey: F9C2E0.” | `f9c2e0` |

The same query returns passive DNS records linking the pseudonym to the original key and documenting its revocation in favor of `F9C2E0`. Querying `kaelen` also returns an archived academic profile identifying Kaelen Voss and the NTNU email `kaelen.voss@alumni.ntnu.no`.

The frontend's evidence and timeline additionally describe the Tromsø Bridge and Arctic Cathedral, a canceled Reykjavik flight, and the Bergen rail disruption. These are challenge-provided records; the solve did not depend on independently validating real-world transit events or verifying a PGP signature.

### Step 2: Reconstruct and verify the flag

This complete Python 3 script uses only the standard library and derives the flag from the starting pseudonym:

```python
import json
import re
from urllib.request import Request, urlopen

base = "https://outdoor-interface-briefing-tests.trycloudflare.com"
with urlopen(base + "/api/search?q=sternenstaub_91", timeout=30) as response:
    records = json.load(response)["results"]

record = next(r for r in records if r["id"] == "snap_threat_intel_leviathan")
metadata = record["metadata"]
name = "_".join(metadata["unmasked_name"].lower().split())
city = metadata["verified_city"].lower()
subkey = re.search(r"hardware subkey:\s*([0-9A-Fa-f]{6})", record["fullContent"])
assert subkey, "Missing genuine hardware subkey"
flag = f"FLAG{{{name}_{city}_{subkey.group(1).lower()}}}"
print(flag)

request = Request(
    base + "/api/submit-flag",
    data=json.dumps({"flag": flag}).encode(),
    headers={"Content-Type": "application/json"},
    method="POST",
)
with urlopen(request, timeout=30) as response:
    result = json.load(response)
print(json.dumps(result, indent=2))
assert result["valid"] is True and result["status"] == "SUCCESS"
```

An actual submission to the mirror returned:

```json
{
  "valid": true,
  "status": "SUCCESS",
  "message": "CHALLENGE CLEARED! Subject Kaelen Voss successfully attributed to Tromsø with authentic subkey F9C2E0. Vanguard-7 CTI dossier updated.",
  "timestamp": "2026-10-10T16:26:27.813Z"
}
```

Tools used: HTTP fetches, curl, and Python 3 standard-library inspection. Saved artifacts: `challenge.js`, `heliopause_evidence_drop.zip`, and `verification.json`. The separate competition scoreboard was not submitted to.

## Flag

```text
FLAG{kaelen_voss_tromso_f9c2e0}
```
