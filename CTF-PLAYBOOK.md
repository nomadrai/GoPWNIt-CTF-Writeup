# CTF Playbook — GoPWNIt

Read this first, every session. Covers what the installed `ctf-skills` don't:
coordination, discipline, and submission — not category tooling, since the
skills auto-trigger for that.

## What's already automatic (don't duplicate this manually)

`~/.claude/skills/` has `ctf-web`, `ctf-pwn`, `ctf-crypto`, `ctf-reverse`,
`ctf-forensics`, `ctf-osint`, `ctf-misc`, `ctf-malware`, `ctf-writeup`,
`ctf-ai-ml`, and `solve-challenge` (the orchestrator — detects category,
routes to the right skill). Claude Code triggers these on its own based on the
challenge description. Trust it to route; don't hand-pick tools unless the
auto-routing clearly picked wrong.

`pwndbg` (MCP) gives Claude Code a live, persistent gdb session for pwn/rev
work — breakpoints, memory inspection, stepping, without losing state
between calls.

## Environment reference (installed tool locations)

- Ghidra: `~/tools/ghidra/ghidraRun`
- pwndbg (real, for pwndbg-mcp): `~/tools/pwndbg`
- Stegsolve: `java -jar ~/tools/stegsolve/Stegsolve.jar`
- Burp Suite CE: `/opt/BurpSuiteCommunity/BurpSuiteCommunity` (if installed — verify)
- jadx: `~/tools/jadx/bin/jadx`
- RsaCtfTool: `~/tools/RsaCtfTool`
- theHarvester: `cd ~/tools/theHarvester && uv run theHarvester -h`
- Everything else (pwntools, radare2, sqlmap, hashcat, john, volatility3,
  sleuthkit, zsteg, etc.) is on PATH — just call it by name.

## Triage protocol (every challenge, before deep-diving)

1. Read title, description, points, attached files/hints.
2. `file <attachment>` on every file — don't guess the type.
3. Let Claude Code's skill auto-routing categorize it. If it guesses wrong
   (e.g. a "web" challenge that's actually crypto underneath), say so
   explicitly and redirect.
4. Rank by points-per-estimated-minute, not by what looks interesting first.
5. **15-minute rule**: stuck past 15 minutes on one challenge with no
   progress -> say so explicitly, move to the next one. Come back later.
   With scoring on points + submission time, a slow correct answer beats a
   stalled one — don't tunnel-vision.

## Two-agent coordination (Claude Code + omnirush)

- Run them on DIFFERENT challenges simultaneously — redundancy against one
  agent's downtime, not duplicated effort on the same problem.
- **Capability asymmetry to track:** Claude Code has `pwndbg` (MCP) and the
  11 `ctf-skills` installed; omnirush's parity depends on whether its own
  MCP/skill registration (in progress) is complete. Until confirmed equal,
  default pwn/rev challenges needing live gdb control to Claude Code, and
  anything omnirush can do with the raw tool stack alone (recon, scripting,
  straightforward crypto/forensics) to omnirush.
- If one agent's session dies or a tool call errors out mid-challenge,
  switch that challenge to the other agent rather than debugging the dead
  session mid-flight.

## Manual flag submission (no auto-submit on this platform)

1. Before submitting, re-read the exploit/tool output yourself — don't
   trust an agent's summary of the flag string blindly. Case, underscores,
   and format wrappers (`flag{...}` vs `CTF{...}`) are easy to get subtly
   wrong in a paraphrase.
2. Copy the flag directly from the tool's raw output, not from the agent's
   prose description of it.
3. Submit immediately once confirmed correct — don't batch submissions,
   since submission time is part of scoring.

## Platform rules (hard constraints, not suggestions)

- Individual participation only — no sharing hints or solutions with other
  participants, in any form, even informally.
- Scanners/fuzzers/automated tools stay scoped to the IP/port/URL given for
  the specific challenge. NEVER point anything at the platform, scoreboard,
  or login infrastructure itself.
- When unsure whether an action counts as "attacking infrastructure,"
  don't do it — skip that approach or ask first.
- Winners are decided by points, then submission time as tiebreaker.
