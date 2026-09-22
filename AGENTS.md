# Makima — HTB Pentest Agent System

This workspace is an automated HackTheBox solving environment. The orchestrator
agent is **makima**; it coordinates specialized sub-agents that each run in
their own Herdr tab (labeled `ai-*`) so work happens in parallel and is visible.

## Environment

- Runs inside **Exegol** (free image) with 400+ pentest tools preinstalled.
- Terminal workspace is managed by **Herdr**; opencode is the primary agent.
- All tools (`nmap`, `netexec`/`nxc`, `impacket`, `certipy`, `bloodhound`,
  `hashcat`, `john`, `sqlmap`, `ffuf`, `nuclei`, `metasploit`, `evil-winrm`,
  `chisel`, `ligolo`, `searchsploit`, etc.) are already on `$PATH`.
- Tool locations, precompiled Windows binaries, webshells, and wordlists are
  documented in the `toolbox` skill. Windows/AD binaries live in
  `/workspace/precompiled-binaries/` (primary) and `/opt/resources/` (secondary).

## Engagement conventions

- One engagement == one HackTheBox machine. Everything for a machine lives in
  `/workspace/engagements/<box-name>/`.
- The canonical notes file is `/workspace/engagements/<box-name>/notes.md`.
  All findings (ports, versions, creds, hashes, flags, next steps) are appended
  there. Sub-agents always read this file first for context and write their
  results back to it.
- **Parallel-safe notes**: each role appends under its own section
  (`## recon`, `## web-exploit`, `## listeners`, …). Append-only — use
  `cat >> notes.md <<'EOF'` or edit only your own section; never rewrite the
  whole file. The orchestrator maintains the top summary.
- **Results travel via files**: sub-agents' terminal output is truncated
  (alternate screen); their real deliverable is what they write to `notes.md`.
  If an orchestrator needs a full report, it asks the agent to write it to a
  file and reply with the path.
- Listener ports are allocated by the orchestrator and recorded under
  `## listeners`; agents never assume a port.
- Create subfolders (`recon/`, `loot/`, `exploits/`, etc.) only when needed,
  never force-create them.

## Herdr orchestration rules

- Sub-agents run in their own tabs labeled `ai-<role>` (e.g. `ai-recon`).
- Live agent names must be unique: `herdr agent start <role>-<box-slug>`.
- Max 4 panes per tab. If a tab needs more concurrent work, create `ai-<role>-2`.
- **The `opencode` tab must never be touched**: never split it, never add panes,
  never close it. All agent work happens in `ai-*` tabs.
- Makima may inspect other (non-`ai-*`) tabs to answer the user's questions
  about their own work, but never modifies them.
- Spawn sub-agents with `--cwd /workspace/engagements/<box-name>` so their work
  lands in the right engagement folder.

## Authorized scope

- Only operate against the target machine the user provided. Do not attack any
  other host, and do not exceed the engagement scope.
- Prefer non-destructive enumeration first, then exploit.
