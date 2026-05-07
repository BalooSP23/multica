# Wiring MCP Servers Into Multica Agents

> Why this guide exists: as of May 2026, Multica's agent-settings GUI exposes only **Environment** (env vars) and **Custom arguments** (CLI args appended to the spawned coding-agent CLI). There is no MCP page yet — [multica-ai/multica#1221](https://github.com/multica-ai/multica/pull/1221) (the "MCP tab" PR) is still open. The DB column `agent.mcp_config` exists ([PR #1168](https://github.com/multica-ai/multica/pull/1168), shipped v0.2.6), but you cannot edit it through the GUI today. This guide is the practical wiring path until that PR lands.

---

## TL;DR

| Provider | How MCP actually loads today | Where you set the config |
|---|---|---|
| **Claude Code** (5 of our 6 agents) | Daemon writes `agent.mcp_config` JSONB to a temp file, then spawns `claude … --mcp-config <tempfile> --strict-mcp-config`. `--mcp-config` is on `claudeBlockedArgs` so Custom arguments **cannot** override it. | Set `agent.mcp_config` via the Multica REST API or `multica` CLI. Secrets referenced as `${VAR}` come from the agent's **Environment** GUI field. |
| **Codex CLI** (Reviewer only) | [PR #1288](https://github.com/multica-ai/multica/pull/1288) (TOML injection into per-task `CODEX_HOME`) is still open as of May 2026 — so `agent.mcp_config` is **silently ignored**. | Pre-populate `~/.codex/config.toml` on the daemon host, or point `MULTICA_CODEX_PATH` at a wrapper script that sets `CODEX_HOME` to a per-agent directory. Custom arguments cannot help (Codex has no `--mcp-config` flag). |

The user's premise — "use Custom arguments + Environment" — is **half right**. Environment is correct for secrets. Custom arguments can NOT currently inject MCP config for either provider. The actual injection happens through the DB column (Claude Code) or daemon-host config files (Codex).

---

## Part A — Claude Code agents (chief-of-staff, backend-builder, frontend-builder, researcher, content-writer)

### A1. Where the JSON lives

The repo ships one `mcp.json` per agent at `agents/<agent-name>/mcp.json`. These are the source-of-truth configs you check into version control. Real secrets are referenced as `${VAR}` and resolved at agent runtime from the **Environment** field.

### A2. Loading the JSON into `agent.mcp_config`

Pick one of three paths, in decreasing order of practicality:

**Path 1 — Multica REST API (recommended, scriptable)**

```bash
# Get a workspace-scoped Personal Access Token from Settings → Account → Tokens (prefix `mul_`).
export MULTICA_TOKEN=mul_xxx
export MULTICA_BASE=http://localhost:8080   # or https://api.multica.ai for cloud

AGENT_ID=$(curl -s -H "Authorization: Bearer $MULTICA_TOKEN" \
  "$MULTICA_BASE/api/v1/workspaces/<slug>/agents" \
  | jq -r '.[] | select(.name=="backend-builder") | .id')

curl -X PATCH \
  -H "Authorization: Bearer $MULTICA_TOKEN" \
  -H "Content-Type: application/json" \
  "$MULTICA_BASE/api/v1/workspaces/<slug>/agents/$AGENT_ID" \
  --data @- <<EOF
{ "mcp_config": $(cat agents/backend-builder/mcp.json) }
EOF
```

The exact path may differ (`/api/v1/agents/{id}` vs `/api/workspaces/{slug}/agents/{id}`) — check `server/cmd/main.go` route registration in your installed version. The PATCH body shape (`mcp_config` as a JSON value, not a string) was added in PR #1168.

**Path 2 — `multica` CLI**

```bash
multica agent update backend-builder --mcp-config-file agents/backend-builder/mcp.json
```

This subcommand is **not confirmed available** in v0.2.6 (CLI_AND_DAEMON.md only documents `multica agent list`). If your installed CLI doesn't have it, fall back to Path 1.

**Path 3 — direct SQL (last resort)**

```sql
UPDATE agent
SET mcp_config = $1::jsonb
WHERE workspace_id = (SELECT id FROM workspace WHERE slug = 'your-slug')
  AND name = 'backend-builder';
```

Only use this if the API and CLI both fail. Do **not** use it in cloud mode.

### A3. Wiring secrets via the Environment field

Multica's **Environment** GUI field stores newline-separated `KEY=VALUE` pairs and injects them as env vars when the daemon spawns the agent CLI. Claude Code expands `${VAR}` and `${VAR:-default}` syntax inside `command`, `args`, `env`, `url`, and `headers` fields of `mcp.json` ([source: Claude Code MCP docs](https://code.claude.com/docs/en/mcp#environment-variable-expansion-in-mcp-json)). So a `mcp.json` line like

```json
"env": { "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_PAT_BACKEND_BUILDER}" }
```

resolves correctly **as long as** `GITHUB_PAT_BACKEND_BUILDER=ghp_xxx` is in that agent's Environment field. If a referenced var is missing and has no default, Claude Code fails to parse the config and the MCP server doesn't load.

### A4. Per-agent `Custom arguments` for Claude Code

Use Custom arguments for non-MCP tuning only. Examples:

```
--model claude-opus-4-5 --permission-mode acceptEdits
```

Do not put `--mcp-config` here — the daemon strips it.

---

## Part B — Codex CLI (reviewer only)

### B1. The injection mechanism Codex needs

Codex CLI reads MCP config from `$CODEX_HOME/config.toml` (default `~/.codex/config.toml`) using `[mcp_servers.NAME]` TOML tables ([source](https://github.com/openai/codex/blob/main/docs/config.md)). There is no `--mcp-config` flag. The `--config / -c key=value` flag can override individual keys but not whole MCP server tables.

### B2. The current Multica gap

Multica's daemon does write a per-task `CODEX_HOME`-like directory for skill injection (see `multica.md` §7 — Codex skills land at `$CODEX_HOME/skills/{name}/`). PR #1288 extends that mechanism to write a `config.toml` with `[mcp_servers.*]` tables from `agent.mcp_config`, but it is still open with reviewer pushback as of April 2026. Until merged, `agent.mcp_config` set on a Codex-runtime agent is silently dropped.

### B3. Workarounds (pick one)

**Workaround 1 — Pre-populate the daemon host's `~/.codex/config.toml`** (simplest)

The Reviewer is the only Codex agent in this roster, so global config is not contended. Create the file once on the daemon machine:

```toml
# ~/.codex/config.toml on the daemon host
[mcp_servers.github]
command = "npx"
args = ["-y", "@modelcontextprotocol/server-github"]
env = { GITHUB_PERSONAL_ACCESS_TOKEN = "ghp_xxx" }

[mcp_servers.sentry]
command = "npx"
args = ["-y", "@sentry/mcp-server"]
env = { SENTRY_AUTH_TOKEN = "snry_xxx" }
```

Secrets are inline (Codex does not auto-expand `${VAR}`). Lock the file's permissions: `chmod 600 ~/.codex/config.toml`.

The TOML version of `mcp.json` lives at `agents/reviewer/mcp.toml` in this repo as the canonical source. Copy/adapt it to the daemon host.

**Workaround 2 — Wrapper script that sets `CODEX_HOME`** (per-agent isolation)

If you ever add a second Codex agent, isolate them with a wrapper:

```bash
#!/usr/bin/env bash
# /usr/local/bin/codex-reviewer
export CODEX_HOME="$HOME/.multica/codex-homes/reviewer"
exec /usr/local/bin/codex "$@"
```

Then on the daemon host: `export MULTICA_CODEX_PATH=/usr/local/bin/codex-reviewer`. Place the agent's `config.toml` at `~/.multica/codex-homes/reviewer/config.toml`. Note that this swaps `CODEX_HOME` globally for the daemon; if you also run Codex outside Multica from the same shell, you'll see your config moved.

**Workaround 3 — Wait for PR #1288 to merge.** Then set `agent.mcp_config` via Path 1 above on the Reviewer agent and Multica handles the TOML injection automatically.

### B4. Per-agent `Custom arguments` for Codex

Use for sandbox + model tuning:

```
--sandbox read-only --model gpt-5.1-codex
```

`MULTICA_CODEX_ARGS` parses with POSIX shellword quoting per `CLI_AND_DAEMON.md`.

---

## Part C — Cross-cutting rules

### C1. Token hygiene (non-negotiable)

Every agent gets its own tokens. Never share a single GitHub PAT, Notion integration secret, or Stripe key across agents.

| Agent | GitHub PAT scope |
|---|---|
| chief-of-staff | `issues:read`, `pull_requests:read`, `metadata:read` (read-only) |
| backend-builder | `contents:write`, `pull_requests:write` (no merge), `issues:read` |
| frontend-builder | `contents:write`, `pull_requests:write` (no merge), `issues:read` |
| researcher | `metadata:read`, `contents:read` (read-only, public repos preferred) |
| content-writer | `contents:write`, `pull_requests:write` — branch-protected `main` blocks merges |
| reviewer | `pull_requests:write` (review comments only), `contents:read`. Explicitly NOT `contents:write`. |

Branch protection on `main` requiring human approval is the last line of defense.

### C2. The Environment field stores plaintext

Per `agent.environment` in the schema, secrets are stored as plaintext JSON in Postgres, encrypted at rest only by whatever DB-level encryption you've configured (none by default in self-host). Implications:

- Use the smallest-scope token that works.
- Rotate quarterly.
- Self-hosters: enable Postgres TDE or full-disk encryption on the DB volume; restrict `daemon_token` and `personal_access_token` rows likewise.
- Do not commit real values to git — the `mcp.json` files in this repo use `${VAR}` placeholders for that reason.

### C3. OAuth-based hosted MCPs (Notion, Vercel, Google Calendar, Figma)

For HTTP-transport MCPs that require OAuth (Notion at `mcp.notion.com/mcp`, Figma at `mcp.figma.com/mcp`, Google Calendar), the `mcp.json` entry is just `{"type":"http","url":"..."}`. The OAuth handshake runs at MCP server startup inside the agent's session — Claude Code's `/mcp` panel triggers it on first connection, and the resulting refresh token is cached in the agent's per-task `.claude/` state. The user must complete the OAuth flow once per agent, per fresh `work_dir`. Multica's session resumption (`multica.md` §5) keeps the same `work_dir` for the same `(agent, issue)` pair, so re-auth is rare in practice. For autonomous Autopilot tasks that lack a human at the keyboard, prefer non-OAuth MCPs (API-key headers).

---

## Part D — Troubleshooting

| Symptom | First check | Likely cause |
|---|---|---|
| Claude agent says "no MCP tools available" | `claude mcp list` inside the running agent's `work_dir` | `agent.mcp_config` is null in DB → set it via Path 1. |
| Agent loads MCPs but a tool call fails with "missing API key" | Agent's Environment field in GUI | Variable referenced by `${VAR}` in `mcp.json` not set in Environment. |
| Codex agent has zero MCP tools regardless of `agent.mcp_config` | Daemon log + `~/.codex/config.toml` on host | Expected — PR #1288 not merged. Use workaround B3.1. |
| `claude` startup hangs > 10s on MCP server connect | `MCP_TIMEOUT=10000 claude ...` env var | Slow stdio MCP startup; raise timeout or switch to HTTP transport. |
| MCP server output exceeds 10k tokens | `MAX_MCP_OUTPUT_TOKENS=50000` in Environment | Verbose tool output truncation warning ([source](https://code.claude.com/docs/en/mcp)). |
| `--mcp-config` set in Custom arguments has no effect | Multica daemon source `claudeBlockedArgs` | Expected — flag is blocked. Use `agent.mcp_config` via API instead. |
| Custom arguments containing spaces parse wrong | POSIX shellword quoting per `CLI_AND_DAEMON.md` | Wrap quoted values: `--model "claude opus 4-5"`. |

---

## Part E — Open questions / not-yet-confirmed

These are documented gaps. Treat them as known-unknowns until a release confirms.

1. **`multica agent update --mcp-config-file` CLI subcommand** — referenced in some community issues but not in official `CLI_AND_DAEMON.md` for v0.2.6. Path 1 (REST API) is the safe fallback.
2. **Multica REST API exact path shape** — `/api/v1/workspaces/{slug}/agents/{id}` vs `/api/v1/agents/{id}`. Verify against your installed version before scripting.
3. **Codex MCP injection (PR #1288) merge timing** — open with reviewer pushback April 2026. When merged, workarounds B3.1 / B3.2 become unnecessary.
4. **MCP tab UI (PR #1221) merge timing** — also open. When merged, all of Part A2 reduces to "paste the JSON in the GUI tab."
5. **Whether `agent.environment` supports `${VAR}` references to the daemon-host's process env** — Multica may store literal values only. If a deployment requires runtime-host env-var reference, test by setting `FOO=${BAR}` in Environment and observing whether the spawned agent sees `FOO=${BAR}` literal or `FOO=<expanded>`. Treat literal-only as the safe assumption.
6. **Whether Multica's `mcp_config` JSONB validates against the Claude Code schema before write** — likely no validation. Bad configs fail at agent runtime, not API write time. Test with `multica agent run-test` (if available) or by assigning a throwaway issue.

---

## Sources

- Multica daemon `--strict-mcp-config` behavior + PR #1168 fix: [multica-ai/multica#1111](https://github.com/multica-ai/multica/issues/1111), [PR #1168](https://github.com/multica-ai/multica/pull/1168)
- MCP UI tab status: [PR #1221](https://github.com/multica-ai/multica/pull/1221) (open as of May 2026)
- Codex MCP injection status: [PR #1288](https://github.com/multica-ai/multica/pull/1288) (open as of April 2026), [Issue #1601](https://github.com/multica-ai/multica/issues/1601)
- Claude Code `--mcp-config` flag + scope precedence: [code.claude.com/docs/en/mcp](https://code.claude.com/docs/en/mcp)
- Claude Code env-var expansion in `.mcp.json`: same docs, "Environment variable expansion" section
- Codex CLI config + `CODEX_HOME` + `[mcp_servers.*]` TOML: [github.com/openai/codex/blob/main/docs/config.md](https://github.com/openai/codex/blob/main/docs/config.md), [developers.openai.com/codex/mcp](https://developers.openai.com/codex/mcp), [developers.openai.com/codex/cli/reference](https://developers.openai.com/codex/cli/reference)
- Multica per-agent override env vars + custom_args precedence: `research/multica.md` §10 + [CLI_AND_DAEMON.md](https://github.com/multica-ai/multica/blob/main/CLI_AND_DAEMON.md)
- Skill-injection-into-work_dir pattern (the model PR #1288 follows): `research/multica.md` §7
