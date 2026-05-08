# Ready to Configure — Multica Agent Roster

> One-page index for the 6 Tier-1 agents. Per-agent canonical content (system prompt, skill list with import URLs, API tools + credentials, anti-scope) lives in each `agents/<name>/` folder. This file points at them and explains the configuration sequence.

## The roster

| Agent | Runtime | One-line role | Folder |
|---|---|---|---|
| **Chief of Staff** | Claude Code (Opus for digests, Sonnet for triage) | Backlog hygiene, daily/weekly digests, dispatch decisions, no code edits. | [chief-of-staff/](./chief-of-staff/) |
| **Backend Builder** | Claude Code (Sonnet default, Opus for hard ones) | Extends API + data model. Migrations before schema, tests before done, PRs not pushes. | [backend-builder/](./backend-builder/) |
| **Frontend Builder** | Claude Code (Sonnet default, Opus for greenfield UI) | Distinctive accessible UIs from workspace tokens. Playwright per interactive flow. PRs not pushes. | [frontend-builder/](./frontend-builder/) |
| **Reviewer** | **Codex CLI** (model diversity vs Builders) | Independent PR review with 5-axis rubric. Comments only — never commits, never merges. | [reviewer/](./reviewer/) |
| **Researcher** | Claude Code (Opus 4.7 1M for synthesis) | Customer / competitive / technical research with primary-source citations. No code, no marketing. | [researcher/](./researcher/) |
| **Content Writer** | Claude Code (Sonnet routine, Opus for launches) | Long-form articles, landing copy, README, launch posts. Drafts only — never auto-publishes. | [content-writer/](./content-writer/) |

Each folder contains:
- `instructions.md` — role, deliverables, system prompt (paste into agent's `system_prompt`), anti-scope, conventions.
- `skills.md` — skill table with skills.sh import URLs + GitHub fallbacks · tools & API access table · per-agent Environment block.

## Architecture choice — API calls, not MCP servers

This roster does **not** use Multica's `agent.mcp_config` JSONB column. Reasons:

- The Multica MCP UI tab is still in-progress ([PR #1221](https://github.com/multica-ai/multica/pull/1221) open as of May 2026); injection paths differ between Claude Code (`--mcp-config`) and Codex (`$CODEX_HOME/config.toml`); env-var expansion is fiddly.
- Every external service the roster needs (GitHub, Supabase, Stripe, Sentry, Notion, Slack, Reddit, Exa, Tavily, Firecrawl) has a well-documented HTTP API or a CLI (`gh`, `supabase`, `stripe`, `multica`, `vercel`, `npx playwright`) that the agent can drive from Bash.
- Credentials live in each agent's **Environment** field (one `KEY=VALUE` per line). The daemon spawns the CLI with these env vars set; the agent's role-specific skill set tells the model which CLI/API to reach for and with what scope.

When PR #1221 lands and the MCP tab ships, this roster can layer MCPs on top of the same env-var set without rewriting agent profiles.

## Configuration sequence

1. **Workspace Context first.** Fill in [`research/WORKSPACE-CONTEXT.template.md`](../research/WORKSPACE-CONTEXT.template.md) for your workspace, then paste the result into Multica → Settings → Workspace → Context. Agents read this on every task; missing it is P0 (especially for Chief of Staff and Reviewer, which can't dispatch / score conventions without it).
2. **Per agent**, in this order: chief-of-staff → backend-builder → frontend-builder → reviewer → researcher → content-writer.
   - Create the agent in Multica → Settings → Agents → New.
   - Pick the runtime listed above.
   - Paste the system prompt from `agents/<name>/instructions.md`.
   - Run the `multica skill import --url <skills.sh URL>` commands listed in `agents/<name>/skills.md`.
   - Fill the agent's **Environment** field with the secrets listed in the "Tools & API access → Wiring in Multica" section of `skills.md`. Each agent has its own GitHub PAT, Notion token, etc. — never share a token between agents.
   - Wire the agent's Autopilots (Chief of Staff has 4; Reviewer has 1 PR-opened webhook; Backend/Frontend/Researcher/Content-Writer have none — they run on issue assignment).
3. **First smoke test.** File a low-stakes issue (e.g. "draft this week's standup digest"), assign to Chief of Staff, watch it run end-to-end.

## Token hygiene (cross-agent)

Every agent gets its own credentials. Never share a single GitHub PAT, Notion token, or Stripe key across agents.

| Agent | GitHub PAT scope |
|---|---|
| chief-of-staff | `issues:read`, `pull_requests:read`, `metadata:read` (read-only) |
| backend-builder | `contents:write`, `pull_requests:write` (no merge), `issues:read` |
| frontend-builder | `contents:write`, `pull_requests:write` (no merge), `issues:read` |
| researcher | `metadata:read`, `contents:read` (read-only, public repos preferred) |
| content-writer | `contents:write`, `pull_requests:write` (no merge), `issues:read` |
| reviewer | `pull_requests:write` (review comments only), `contents:read`. Explicitly NOT `contents:write`. |

Branch protection on `main` requiring human approval is the last line of defense.

## Marketplace fallback chain

1. **`skills.sh`** — primary. All 56 skill URLs across this roster are verified live.
2. **GitHub** — every `skills.md` lists a fallback GitHub URL per skill. `multica skill import --url <github-tree-url>` works for any public repo containing a `SKILL.md`.
3. **`officialskills.sh`** — third option for vendor-official skills (anthropics, vercel-labs, stripe, supabase, trailofbits). URL pattern: `https://officialskills.sh/<org>/skills/<skill>` (single `skills` segment).

**`clawhub.ai`** is a community marketplace referenced by Multica's docs but is small (~50 skills); none of this roster's canonical skills are mirrored there. Last-resort fallback only.
