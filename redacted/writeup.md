---
title: "redacted"
ctf: "GoPWNIt CTF 2026"
date: 2026-10-10
category: osint
difficulty: easy
points: 50
flag_format: "GPI{...}"
author: "nomadrai"
---

# redacted

## Summary

A visually redacted briefing retains deleted text, hidden corrections, and an
unedited image thumbnail. The existing solver correlates those remnants with
a staff directory to supply the three inputs required by the answer checker.

## Solution — probable approach

This reconstruction follows [solve.py](solve.py) and the extracted document
structure. The final badge selection and checker output were not retained.

### 1. Inspect the document package rather than its visible page

Open `dist/briefing.docx` as a ZIP and inspect `word/document.xml`. The saved
script retrieves the source identity from a tracked-deletion element, `w:del`,
and reads hidden paragraphs containing `w:vanish`. A paragraph containing
`Correction` supplies the corrected gate, matched as a letter and number.

Black overlays, tracked deletions, and hidden formatting do not remove the
underlying XML. Preserve the distinction between the deleted original identity
and the later hidden logistical correction.

### 2. Correlate the retained thumbnail with the staff directory

The script reads `word/media/image1.jpg` and carves the embedded JPEG between
its second start marker and the following end marker. The existing
`thumbnail.jpg` is the retained output of this step. Its original badge is the
intended lookup value for `dist/staff_directory.csv`.

Match the badge exactly and require a unique row, then obtain that row's
`full_name`. The badge value itself is not transcribed in the saved solver,
so this writeup does not invent a name or badge number.

### 3. Supply the correlated fields to the checker

The retained script passes the original source, matched staff name, and
corrected gate to `dist/unlock.py`. This documents the expected completion
path without performing a fresh extraction or checker execution.

## Tools and lesson

Python's ZIP, XML, CSV, and byte-processing libraries implement the saved path.
Effective redaction must remove historical text and embedded previews as well
as changing the rendered document.

## Flag

**Submission note (participant-provided):** The flag was not saved at the moment
of submission due to hurry.
