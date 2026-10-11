---
title: "GoPwnIt CTF Participant Feedback"
ctf: "GoPWNIt"
date: 2026-10-10
category: misc
difficulty: easy
points: 50
flag_format: "GPI{...}"
author: "nomadrai"
---

# GoPwnIt CTF Participant Feedback

## Summary

The challenge links to a Google feedback form. Its public HTML includes the
submission-confirmation message in `FB_PUBLIC_LOAD_DATA_`, revealing the flag.

## Solution

Open the form linked by the challenge:

<https://forms.gle/nD9tZLtm8c3bdxDR8>

Inspect its HTML source and search for `GPI{`. The embedded
`FB_PUBLIC_LOAD_DATA_` array contains:

```text
Here is your feedback flag:
GPI{y0ur_f33db4ck_m4tt3rs}
```

The complete runnable [solve.py](solve.py) fetches the public form page with
`curl` and extracts the flag using Python's standard library:

```bash
python "GoPwnIt CTF Participant Feedback/solve.py"
```

## Flag

```text
GPI{y0ur_f33db4ck_m4tt3rs}
```

Submitted to `gopwnit-ctf-participant-feedback-87a116` and accepted for
**50 points, #8 Solve**. The challenge was independently confirmed in the
account's solved list. See [submission_result.json](submission_result.json).
