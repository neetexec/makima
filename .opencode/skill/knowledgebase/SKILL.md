---
name: knowledgebase
description: How to navigate the pentest knowledgebase (3 repos under /workspace/knowledgebase) efficiently using grep/glob/read. Use when you need a payload, technique, or command reference for any pentest task.
---

# Knowledgebase navigation

The knowledgebase lives at `/workspace/knowledgebase/` with three repos. Do NOT
read files blindly — search first, then read only the relevant file.

**Always start from `/workspace/knowledgebase/INDEX.md`** — it maps every topic
to the exact file path, so you jump straight to the right file instead of
grepping across 954 files.

**Fallback:** if the KB does not cover what you need (niche box, new CVE, custom
app), do not force it — use the installed tools, general pentest knowledge, and
web research, and note in `notes.md` that the KB did not cover it.

## What's where

| Repo (path) | What it's good for |
|---|---|
| `knowledgebase/PayloadsAllTheThings` | Web/API exploit payloads: 64 vuln categories (SQLi, XSS, SSTI, SSRF, XXE, LFI/RFI, file upload, deserialization, JWT, NoSQL, GraphQL, command injection, etc.). Each category has `README.md` (+ `Files/`, `Intruder(s)/`, `Images/`). |
| `knowledgebase/InternalAllTheThings` | Internal/AD & post-exploitation: `active-directory/` (enum + exploit), `redteam/escalation/` (Linux & Windows privesc playbooks), `cheatsheets/` (shells, hash cracking, mimikatz, network discovery), `containers/` (Docker/K8s), `databases/` (MSSQL), `redteam/persistence|pivoting|evasion`. |
| `knowledgebase/HackerRecipes` | The Hacker Recipes: `ad/` (recon/movement/persistence), `web/` (recon/inputs/config), `infra/` (pivoting, protocols, privesc). |

## How to search

1. **Find the right file** — `glob` for names, `grep` for keywords:

```
grep -ri "kerberoast" /workspace/knowledgebase/InternalAllTheThings/active-directory
grep -rl "SSTI" /workspace/knowledgebase/PayloadsAllTheThings
glob "**/sudo.md"  # or use ls/glob within the kb path
```

2. **Read the exact file** (not the whole repo). Typical entry points:
   - Payload category → `README.md`; per-DBMS/per-language detail may be split
     into sibling `.md` files (e.g. SQLi has `MySQL Injection.md`,
     `MSSQL Injection.md`).
   - Ready-to-use artifacts are in `Files/` (shells, `.htaccess`, `.xsl`,
     `.svg`, images) and wordlists in `Intruder(s)/`.
3. **Cross-check tools** — the KB references tools that are already installed in
   Exegol (`nmap`, `netexec`, `impacket`, `certipy`, `hashcat`, `ffuf`,
   `sqlmap`, `nuclei`, `bloodhound.py`, etc.). Run `which <tool>` if unsure.

## Repository cross-reference (same topic in multiple repos)

- AD: `InternalAllTheThings/active-directory` (cheatsheet commands) +
  `HackerRecipes/ad` (theory + full walkthroughs). Read both for deep attacks.
- Privesc: `InternalAllTheThings/redteam/escalation` is the *authoritative*
  playbook; `HackerRecipes/infra/privilege-escalation` is thinner.
- Web exploit: `PayloadsAllTheThings` is authoritative for payloads;
  `HackerRecipes/web/inputs` gives theory + tool commands.

## Gotchas

- Many files in HackerRecipes `windows privesc`, `infra/protocols`, and a few
  IATT stubs are near-empty. Do not rely on them; use the authoritative files
  listed above instead.
- `PayloadsAllTheThings/Methodology and Resources` is a cross-cutting umbrella
  (recon/shells/privesc/AD cheatsheets), not a single vuln.
