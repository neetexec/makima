---
description: Start solving a HackTheBox machine. Usage: /solve <ip> [box-name]
agent: makima
---

Start a new engagement for the target `$ARGUMENTS`.

1. Determine the target IP (and optional box name). If a box name is not given,
   derive one from the IP (e.g. `10.10.11.x`) or ask the user.
2. Create `/workspace/engagements/<box-name>/notes.md` with a header: target,
   box name, start time.
3. Begin the HTB methodology (see the `htb-methodology` skill): dispatch `recon`
   (and `web-recon` if HTTP is indicated) into `ai-*` tabs, then proceed phase
   by phase until both flags are captured.
4. Report progress and flags to the user as you go.
