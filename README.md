# GoPWNIt CTF 2026

A collection of challenge writeups from **GoPWNIt CTF 2026**, documenting the
available solution scripts, investigation notes, and retained local artifacts.
The repository covers all **44 challenges**, with one `writeup.md` in each
challenge directory.

**[Browse all 44 writeups →](WRITEUPS.md)**

## About the event

GoPWNIt CTF 2026 was an online, individual Capture the Flag competition focused
on offensive security, digital investigation, and problem-solving. Participants
worked through challenges ranging from introductory puzzles to multi-stage
cryptographic, forensic, and reverse-engineering tasks.

| Detail | Information |
|---|---|
| Event | GoPWNIt CTF 2026 |
| Platform | [gopwnit.com](https://gopwnit.com/) |
| Date | 10 October 2026 |
| Competition window | 12:30–18:30 UTC — six hours |
| Participation | Individual |
| Challenges | 44 |
| Difficulty distribution | 14 easy · 17 medium · 13 hard |
| Event page | [Challenge dashboard](https://gopwnit.com/dashboard/seasons/gopwnit-ctf-2026-1b1bc8/challenges) |

## Repository overview

The writeups describe each challenge's central idea, relevant evidence, and
solution process or reconstructed probable approach. Supporting files are kept
locally with their challenge. Git includes each challenge's `writeup.md` and
ignores its other files and nested directories, as defined in [.gitignore](.gitignore).
References to those supporting artifacts describe the local competition workspace.

The collection includes work on RSA and ECDSA weaknesses, timing leakage,
encrypted archives, image and audio steganography, memory and network evidence,
binary validation routines, document recovery, OSINT, and web application trust
boundaries.

### Challenge categories

Categories below follow the event's dashboard. Some writeup metadata uses a
broader technical category—for example, forensic analysis for a steganography
challenge.

| Platform category | Challenges |
|---|---:|
| Cryptography | 9 |
| Forensics | 7 |
| Miscellaneous | 7 |
| OSINT | 5 |
| Other | 1 |
| Reverse Engineering | 5 |
| Steganography | 4 |
| Web Exploitation | 6 |
| **Total** | **44** |

### Layout

```text
.
├── README.md                 Event information and repository overview
├── WRITEUPS.md               Complete challenge index
├── CTF-PLAYBOOK.md           Competition workflow and tooling reference
├── .gitignore               Writeup-only rules for challenge directories
└── <challenge directory>/
    ├── writeup.md            Challenge explanation
    ├── solve.py              Local solution script, where available; ignored
    └── ...                   Local attachments, evidence, and outputs; ignored
```

The exact contents vary by challenge. Some directories contain full offline
handouts and scripts; others contain a writeup reconstructed from the published
description. Original directory names are retained, including spaces. The links
in [WRITEUPS.md](WRITEUPS.md) resolve to those exact names.

## Reading the writeups

1. Open the [challenge index](WRITEUPS.md) to choose a challenge by category,
   difficulty, or points.
2. Read its `writeup.md` for the key observations, approach, and flag record.
3. Consult the neighboring artifacts and scripts for implementation details.
   Inputs, dependencies, working directories, and validation requirements are
   documented individually where available.

### Flag and evidence records

- **25 writeups include a locally retained flag record.** The source may be a
  saved plaintext file, an earlier writeup, or another retained artifact.
  Individual documents distinguish local verification from saved platform
  acceptance evidence.
- **19 writeups include a submission note** recording that the final flag or
  key was not saved at the moment of submission due to hurry. Their probable
  approaches are explicitly labeled and grounded in the available scripts,
  handouts, or challenge descriptions.

The repository preserves the available competition material and its provenance.
The post-event documentation pass added the missing writeups without running
new solves or making new flag submissions.

## Event and challenge credits

The event, challenge designs, and original handouts belong to their respective
organizers and challenge authors. Author credits and evidence references are
retained in the individual writeups and supplied materials.
