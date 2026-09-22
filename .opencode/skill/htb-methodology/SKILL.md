---
name: htb-methodology
description: Overall strategy for solving a HackTheBox machine end-to-end (recon -> foothold -> privesc -> flags), flag conventions, and note-taking discipline. Use when starting a new engagement or deciding next steps.
---

# HTB methodology

The shared playbook for solving a HackTheBox machine. Use this to keep every
phase ordered and every agent writing to the same source of truth.

## Phases

1. **Recon** — enumerate ports/services/OS (`recon`) and, if HTTP is present, the
   web surface (`web-recon`) in parallel.
2. **Analyze** — from the attack surface, pick the path:
   - versioned service → `cve-research` (researches AND executes validated PoCs —
     it is the executor for non-web service exploits);
   - web app → `web-exploit`;
   - AD indicators (SMB 445/139, Kerberos 88, LDAP 389/636) → `ad-enum`.
3. **Foothold** — turn the vuln into RCE/shell (`web-exploit`, `cve-research`,
   `persistence` for shells/TTY). Record creds/hash.
4. **Privesc** — `linux-privesc` or `windows-privesc` by OS; on AD boxes follow
   with `ad-exploit` for lateral movement / domain compromise.
5. **Flags** — capture `user.txt` then `root.txt` (locations below).
6. **Report** (optional) — a short chain summary in `notes.md`.

## Flag conventions

- Linux: `/home/<user>/user.txt`, `/root/root.txt`.
- Windows: `C:\Users\<user>\Desktop\user.txt`,
  `C:\Users\Administrator\Desktop\root.txt`.
- HTB flags look like a 32-char hex string. Record them verbatim and immediately.

## Flag verification

- A reported flag must be re-read from its file (with full path) in the same
  report — never accepted from memory alone.
- Validate format `^[0-9a-f]{32}$`. The orchestrator records it as confirmed
  only after this check.

## Note-taking discipline (parallel-safe)

- Canonical file: `/workspace/engagements/<box-name>/notes.md`.
- **Each role appends under its own section** (`## recon`, `## web-recon`,
  `## web-exploit`, `## listeners`, …) so parallel agents never overwrite each
  other. The orchestrator maintains the top summary.
- **Append-only**: use `cat >> notes.md <<'EOF' ... EOF` or an Edit that targets
  your own section. NEVER rewrite the whole file.
- Before each phase, read `notes.md`. After each result, write it back.
- Subfolders (`recon/`, `loot/`, `exploits/`) are created only when needed.

## CVE checks (supplement)

- The orchestrator dispatches the persistent `cve-research` agent when a
  reported version looks like a plausible attack vector (web framework/CMS,
  standout service, kernel/sudo during privesc).
- Boring standard services (SSH banner, generic web-server versions) are
  skipped unless something specific points at them.
- CVE checks are a supplement to the main recon/exploit flow — never a blocker:
  dispatch and keep moving; results land in the `## cve-research` notes section
  and are acted on when useful.

## Handoff rules

- Pass precise context when dispatching an agent: target, current user/creds,
  relevant ports, and a pointer to `notes.md`.
- Never have two agents mutate the same thing (same listener port, same relay
  target) simultaneously.
- Keep long-running work (cracking, fuzzing) in its own pane/tab.

## Using the knowledge base (optional, not required)

- The knowledgebase (`/workspace/knowledgebase/`) is *one* source, not the only
  source. Consult it via the `knowledgebase` skill when it applies.
- The KB does not cover everything. When a scenario falls outside it, fall back
  to general knowledge, installed tool output, and web research. Do not force a
  KB match, and do not stall just because the KB lacks a page.
- Agents should record which KB file they used (or note "not in KB") in
  `notes.md` for traceability.

## Scope & safety

- Only the user-provided target. No other host. Non-destructive enumeration
  first, then exploit. Avoid destructive actions (encrypting, wiping, dropping
  persistence) unless the box/instructions require it.
