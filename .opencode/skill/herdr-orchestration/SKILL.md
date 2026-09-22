---
name: herdr-orchestration
description: Makima conventions for orchestrating pentest sub-agents in Herdr. Use when spawning/coordinating ai-* tabs and panes, running parallel scans, or inspecting the user's tabs. Builds on the base 'herdr' skill.
---

# Herdr orchestration (Makima)

This skill adds the HTB-orchestration conventions on top of the base `herdr`
skill. Read that skill first for the full CLI surface.

## When you are the orchestrator (makima)

You use Herdr automatically to run the engagement. You are NOT limited to
"when the user mentions Herdr" — spawning sub-agents in `ai-*` tabs is your
normal way of working.

## Sourcing the session (works even when HERDR_ENV is not injected)

The orchestrator's opencode session may NOT have `HERDR_ENV`/`HERDR_WORKSPACE_ID`
injected (verified in practice). The herdr CLI still works via its socket.
Therefore:

```bash
if [ "${HERDR_ENV:-}" = 1 ]; then
  WS="$HERDR_WORKSPACE_ID"                       # injected context
else
  WS=$(herdr workspace list | python3 -c "import sys,json;d=json.load(sys.stdin);print(d['result']['workspaces'][0]['workspace_id'])")
fi
test -n "$WS" || { echo "no herdr workspace found"; return 1; }
```

- Pass `--workspace "$WS"` explicitly on every `tab create`.
- Use `--current`/injected pane ids only when `HERDR_ENV=1`.
- Sub-agents spawned by herdr DO get `HERDR_ENV` injected into their panes.
- Herdr is always present. If the CLI ever fails, stop and report the error to
  the user — do not invent an alternate execution path.

## Tab naming & topology

- Sub-agent tabs are named `ai-<role>`: `ai-recon`, `ai-web-recon`,
  `ai-web-exploit`, `ai-cve-research`, `ai-ad-enum`, `ai-ad-exploit`,
  `ai-linux-privesc`, `ai-windows-privesc`, `ai-persistence`.
- **Live agent names must be unique** across the whole session. Use
  `herdr agent start <role>-<box-slug> ...` (e.g. `recon-support`), never a bare
  role name, so a second engagement or a respawn cannot collide. Tab labels stay
  `ai-<role>`.
- Max **4 panes per tab**. Need more parallel work in the same role? Create a
  new tab `ai-<role>-2`, `ai-<role>-3`, etc.
- **The `opencode` tab is off-limits.** Never split it, add panes to it, or close
  it. All your spawned work lives in `ai-*` tabs.

## Spawn a sub-agent in its own tab (persistent)

```bash
# create a new tab (NOT a split of the opencode tab), cwd = engagement folder
herdr tab create \
  --workspace "$WS" \
  --cwd /workspace/engagements/<box-name> \
  --label ai-recon \
  --no-focus
# -> read ".result.root_pane.pane_id" (and ".result.tab.tab_id")

herdr agent start recon-<box-slug> --kind opencode --pane <pane-id> -- --agent recon
herdr agent wait  recon-<box-slug> --until idle --timeout 60000
herdr agent prompt recon-<box-slug> "<task with full context>" --wait --timeout 300000
herdr agent wait  recon-<box-slug> --until idle --timeout 300000
```

Notes:
- `agent start` waits for the agent to be ready (default 30s). If it returns
  `agent_not_ready`, `agent wait --until idle` before prompting.
- **First prompt after start is often swallowed** (`agent_prompt_stalled`) while
  the TUI finishes rendering (verified). After a stall: `agent wait --until
  idle`, then re-send the same prompt once. It succeeds the second time.
- Pass full context in the prompt (target, creds, findings pointer) — the
  sub-agent's context is fresh.
- `agent prompt --wait` waits for the next settled `idle`/`done`/`blocked`.
  If `blocked`, inspect with `agent read` and ask the user before answering an
  approval/question dialog.

## Results come from FILES, not from terminal reads

opencode's TUI uses the alternate screen; `agent read` only sees the last
viewport and CANNOT recover earlier output. Therefore:

- **Treat `/workspace/engagements/<box-name>/notes.md` as the real result.**
  Sub-agents are instructed to write their findings there. Read that file with
  your own read tool to collect results.
- Use `agent read` only as a quick liveness/progress check.
- If you still need a full report from an agent and the terminal is truncated:
  ask the agent to write its complete response as Markdown into a file under the
  engagement folder and reply with just the file path, then read the file.

## Retry, poll, and timeouts

- `agent prompt` fails with `agent_prompt_stalled` if the agent does not visibly
  start within ~5 s — that does NOT mean the work failed. `agent wait` for
  `idle|done|blocked` and then check `notes.md`.
- Phases take longer than 5 minutes. Use per-phase timeouts:
  - recon / web-recon / ad-enum: 900000 ms
  - web-exploit / ad-exploit / privesc / cve-research: 600000 ms
- For long phases, do NOT block on one prompt forever. Poll the engagement
  `notes.md` for progress; if the agent settles early with partial work, send a
  follow-up prompt: "Continue where you left off; report progress to notes.md."
- If `agent start` times out, `agent wait --until idle` before the first prompt.

## Persistent helper tab (cve-research pattern)

Some roles are long-lived helpers you spawn once per engagement and prompt on
demand. `cve-research` is the model:

1. Spawn once: tab `ai-cve-research`, agent `cve-research-<box-slug>`.
2. Prompt it whenever you want a version/service checked, e.g.
   "Check <software> <version> @ <target> for applicable CVEs; write findings
   to `notes.md` under `## cve-research`."
3. **Non-blocking**: never wait on it — read its `## cve-research` notes section
   the next time you read `notes.md`.
4. It idles between requests — that is normal. Re-prompt when you have a new
   version, and don't re-send one you already sent.

## Listener ports (coordination)

- The orchestrator allocates listener ports per engagement and records them in
  `notes.md` (section `## listeners`), e.g. web shell 9001, privesc 9002, relay
  9003. Sub-agents must use the assigned port, never assume 4444.

## Run parallel raw commands inside a tab

Both you and sub-agents can add panes to a tab for long-running/parallel tool
commands (max 4 panes total per tab):

```bash
herdr pane split --pane <existing-pane-id> --direction right \
  --cwd /workspace/engagements/<box-name> --no-focus
# -> ".result.pane.pane_id"
herdr pane run <new-pane-id> "ffuf -u http://TARGET/FUZZ -w /usr/share/seclists/Discovery/Web-Content/raft-medium-words.txt"
herdr pane wait-output <new-pane-id> --match "..." --timeout 120000
herdr pane read <new-pane-id> --source recent-unwrapped --lines 200
```

## Inspect the user's own tabs (read-only, never modify)

To answer the user's questions about their work, inspect non-`ai-*` tabs only:

```bash
herdr tab list
herdr agent list
herdr api snapshot                # full live state
herdr pane read <pane-id> --source recent-unwrapped --lines 200
herdr agent read <agent-name> --source recent-unwrapped --lines 200
```

Never `agent prompt`, `send-keys`, `pane run`, or `close` the user's tabs/panes.
Only `ai-*` tabs you created may be prompted or closed.

## Cleanup

- Close only `ai-*` tabs you created: `herdr tab close <tab-id>`.
- Do not close workspaces/tabs/panes you did not create, and never stop the
  Herdr server.

## Tracking

- Every sub-agent writes findings to
  `/workspace/engagements/<box-name>/notes.md`. Read it before each dispatch and
  after each result — it is the source of truth, not the terminal.
