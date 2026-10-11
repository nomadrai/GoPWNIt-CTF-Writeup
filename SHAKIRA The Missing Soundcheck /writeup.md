---
title: "SHAKIRA The Missing Soundcheck"
ctf: "GoPWNIt CTF 2026"
date: 2026-10-10
category: osint
difficulty: hard
points: 150
flag_format: "GPI{...}"
author: "nomadrai"
---

# SHAKIRA The Missing Soundcheck

## Summary

The fictional offline archive misattributes a rehearsal photograph to Forum del
Sol. Architectural and transit evidence identifies Coliseo Marina Sur; current
access plans, an acoustic inspection log, and the municipal booking ledger supply
the remaining archive-index fields.

## Solution

### 1. Identify the venue and current entrance

The original photographer's post is `archived_posts/post_archive_7995.html`, by
Lucia Mendez (`@lens_art_live`), with production identifier `TOUR-SK26-PRD`.
Post `8119` falsely labels the venue Forum del Sol. Technician post `8144`
challenges that attribution using the roof and canal bridge.

The master image `photographs/plate_01_rehearsal_master.png` shows:

- Asymmetric, parabolic steel roof arches.
- Upper clerestory windows facing Pasarela Norte N-3 and the Cerro Faro masts.
- A yellow-and-black artist/technical entrance explicitly labeled `STAGEB`.

The two architectural comparison plates, district master plan, transit guide,
and venue directory uniquely match **Coliseo Marina Sur, code `COLMAR5`**.
Forum del Sol has a flat triangular space-frame roof and no canal skybridge.

The current logistics plan, revision 2.0, and the DSS-26 reclassification circular
confirm **`STAGEB`** as the artists/soundcheck portal from 1 September 2026.
`GATE2` is the obsolete designation; `DOCKA` serves heavy rigging trucks.

### 2. Recover the actual start time and booking

The preliminary itinerary's 18:30 AST soundcheck was canceled by operations
bulletin 04. That bulletin's 21:15–21:30 interval is only standby.

`production_notices/sound_compliance_log_20261018.pdf` certifies the actual
soundcheck start at **21:40 AST on 18 October 2026**, measured at **91.2 dBA**.
The photograph's acoustic monitor shows the same reading. The original post's
22:05 AST timestamp is publication time, not the event's start.

`public_records/district_time_and_standards_gazette.pdf` specifies:

- AST is fixed at UTC−04:00, without daylight-saving changes.
- `LOCAL_DATE` is the venue's civil date in `YYYYMMDD`.
- `UTC_TIME` is the converted clock time in `HHMM`.
- Midnight rollover does **not** replace `LOCAL_DATE` with the UTC date.

Therefore:

```text
2026-10-18 21:40 AST = 2026-10-19 01:40 UTC
DATE = 20261018
TIME_UTC = 0140
```

The municipal operations ledger's matching row is **`BK9427`**:
`COLMAR5`, local date `2026-10-18`, El Dorado Live Touring Corp,
“Ensayo de Gira y Soundcheck Oficial (World Tour).”
`BK9554` covers the next day's public concert instead.

### 3. Assemble and verify

The live challenge description specifies underscores:

```text
[VENUE]_[PORTAL]_[DATE]_[TIME_UTC]_[REF]
```

| Field | Value |
|---|---|
| VENUE | `COLMAR5` |
| PORTAL | `STAGEB` |
| DATE | `20261018` |
| TIME_UTC | `0140` |
| REF | `BK9427` |

The accompanying `solve.py` extracts the relevant PDF records directly from the
original ZIP, computes the timezone conversion, and selects the matching ledger
entry. Run from the project root:

```bash
uv run --no-project --with pymupdf python "SHAKIRA The Missing Soundcheck/solve.py"
```

The live checker accepted the submission on **2026-10-10 at 16:53:22 UTC**:

```json
{"message": "Flag accepted.", "rank": "#12 Solve", "points": 150}
```

## Flag

```text
GPI{COLMAR5_STAGEB_20261018_0140_BK9427}
```
