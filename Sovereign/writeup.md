---
title: "Sovereign"
ctf: "GoPWNIt"
date: 2026-10-10
category: osint
difficulty: medium
points: 100
flag_format: "GPI{...}"
author: "nomadrai"
---

# Sovereign

## Summary

The supplied JPEG's EXIF comment explicitly contains all four recovery fields.
The reverted XML revision independently confirms them. Lowercasing the antenna
code and island name, and removing antenna-code punctuation, produces the flag
accepted by the live checker for 100 points.

## Solution

### 1. Recover the required fields

`incident_report.txt:32` specifies:

```text
GPI{<antenna_code>_<target_island>_<incident_year>_<hex_auth>}
```

Reading the JPEG with Pillow's `Image.open(...).getexif()` reveals this
`UserComment`, also visible using `strings -n 8 survey_station_alpha.jpg`:

```text
STATION: AN/FLR-9 | TARGET: BOUVET | EVENT: 1979 | HASH: 8f2c9a
```

The image itself names the same antenna, Bouvet Island, the Vela event in 1979,
and telemetry hash `8f2c9a`. Its EXIF `ImageDescription` corroborates the station,
island, and authorization value. No GPS IFD is present despite the briefing's
claim about EXIF GPS coordinates.

### 2. Corroborate with the reverted revision

`wikipedia_history_dump.xml:31`, revision `7419082`, contains:

```text
In September 1979 ...
id=Q31057:P625:WULLENWEBER
Long-range High-Frequency Direction Finding (HFDF) station AN/FLR-9 ...
Hexadecimal authorization crumb: 8f2c9a
```

The page title and article text identify Bouvet Island. Revision `7419114`
removes the relevant passage, explaining why the earlier revision is needed.

External checks confirm that AN/FLR-9 is an HF direction-finding antenna array
and the Vela incident occurred on 22 September 1979. The challenge's Q-ID is
incorrect: Q31057 is Norfolk Island; Bouvet Island is Q23408. The repeated
explicit target in the supplied evidence is therefore the useful flag field.

### 3. Reproduce extraction

The adjacent `solve.py` uses only Python's standard library. It reads the
original ZIP, extracts the metadata fields, corroborates them against the XML,
and prints the accepted flag:

```bash
python Sovereign/solve.py
```

## Flag

```text
GPI{anflr9_bouvet_1979_8f2c9a}
```

The attachment does not specify capitalization or punctuation normalization.
The uppercase versions were rejected; the lowercase, alphanumeric antenna-code
version above was accepted on 2026-10-10 at 16:45:23 UTC:

```json
{"message": "Flag accepted.", "rank": "#11 Solve", "points": 100}
```

## Sources

- Supplied `incident_report.txt`, `wikipedia_history_dump.xml`, and JPEG EXIF.
- [AN/FLR-9](https://en.wikipedia.org/wiki/AN/FLR-9).
- [Vela incident](https://en.wikipedia.org/wiki/Vela_incident).
- [Q31057: Norfolk Island](https://www.wikidata.org/wiki/Q31057).
- [Q23408: Bouvet Island](https://www.wikidata.org/wiki/Q23408).
- [P625: coordinate location](https://www.wikidata.org/wiki/Property:P625).
