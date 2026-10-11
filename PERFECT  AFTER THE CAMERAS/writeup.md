---
title: "PERFECT AFTER THE CAMERAS"
ctf: "GoPWNIt CTF 2026"
date: 2026-10-10
category: osint
difficulty: easy
points: 50
flag_format: "GPI{VENUENAME_YYYYMMDD_TICKETS_COLOUR}"
author: "nomadrai"
---

# PERFECT AFTER THE CAMERAS

## Summary

The challenge follows a song from its lyrics to its official winter-set music
video, then to a later exclusive concert at the same après-ski venue. The
answer combines the venue, actual event date, worldwide ticket allocation,
and cocktail colour.

## Solution — probable approach

The saved basis is the dashboard prompt and starting video,
`https://youtu.be/5A5wkqhCMTU`. Research notes and the four final values are not
retained. This describes the intended source-correlation process without
performing new OSINT research.

### 1. Establish the song and filming venue

Identify the song from the starting clip, then locate the artist's official
narrative video rather than relying on a lyric upload or fan re-edit. Use
official filming credits, venue publicity, or production coverage to connect
its winter setting to the named après-ski venue.

Maintain that venue link explicitly: a ski resort, hotel, and bar can appear
in coverage of the same production but are not interchangeable flag fields.

### 2. Find coverage of the later exclusive concert

Search the venue's announcements and event recaps for the artist's return.
Extract the concert's actual date from event coverage, distinguishing it from
the article publication date and the original video shoot. Corroborate the
number of limited tickets distributed worldwide and the colour of cocktails
served that evening using the concert's own publicity or reporting.

The ticket field asks about distribution, not venue capacity or attendance.
Likewise, the requested colour should come from the drinks described at that
event, not a guess based on a sponsor's branding.

### 3. Normalize the four fields

Preserve the documented venue spelling, express the event date as `YYYYMMDD`,
and serialize the values in the supplied order:

```text
GPI{VENUENAME_YYYYMMDD_TICKETS_COLOUR}
```

This is the published format, not a recovered flag.

## Tools and lesson

Probable tools: the official video, search engines, venue archives, and event
coverage. Linking two separate events requires careful date and entity
resolution; a publication timestamp or capacity figure is an easy false match.

## Flag

**Submission note (participant-provided):** The flag was not saved at the moment
of submission due to hurry.
