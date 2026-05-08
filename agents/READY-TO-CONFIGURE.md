# Ready to Configure — Multica Agent Roster

> One-page index for the 6 Tier-1 agents. Per-agent canonical content (system prompt, skill list with import URLs, MCP config, anti-scope) lives in each `agents/<name>/` folder. This file points at them and explains the configuration sequence.

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
- `skills.md` — skill table with skills.sh import URLs + GitHub fallbacks, MCP server table, wiring notes.
- `mcp.json` (or `mcp.toml` for Reviewer) — the `agent.mcp_config` content.

## Configuration sequence

1. **Workspace Context first.** Fill in [`research/WORKSPACE-CONTEXT.template.md`](../research/WORKSPACE-CONTEXT.template.md) for your workspace, then paste the result into Multica → Settings → Workspace → Context. Agents read this on every task; missing it is P0 (especially for Chief of Staff and Reviewer, which can't dispatch / score conventions without it).
2. **Per agent**, in this order: chief-of-staff → backend-builder → frontend-builder → reviewer → researcher → content-writer.
   - Create the agent in Multica → Settings → Agents → New.
   - Pick the runtime listed above.
   - Paste the system prompt from `agents/<name>/instructions.md`.
   - Run the `multica skill import --url <skills.sh URL>` commands listed in `agents/<name>/skills.md`. (Only attached skills load into the agent's task `work_dir`.)
   - Load the per-agent `mcp.json` (or `mcp.toml`) into `agent.mcp_config` per [`MCP-WIRING.md`](./MCP-WIRING.md). The Multica GUI does not yet have an MCP tab (PR #1221 open) — use the REST API path documented there.
   - Fill the agent's **Environment** field with the secrets referenced as `${VAR}` in the MCP config (per-agent token table in `MCP-WIRING.md` Part C1).
   - Wire the agent's Autopilots (Chief of Staff has 4; Reviewer has 1 PR-opened webhook; Backend/Frontend/Researcher/Content-Writer have none — they run on issue assignment).
3. **First smoke test.** File a low-stakes issue (e.g. "draft this week's standup digest"), assign to Chief of Staff, watch it run end-to-end.

## Marketplace fallback chain

1. **`skills.sh`** — primary. All 56 skill URLs across this roster are verified live (HEAD-checked 2026-05-08).
2. **GitHub** — every `skills.md` lists a `# fallback:` GitHub URL per skill. `multica skill import --url <github-tree-url>` works for any public repo containing a `SKILL.md`.
3. **`officialskills.sh`** — third option for vendor-official skills (anthropics, vercel-labs, stripe, supabase, trailofbits). Same SKILL.md, different host. URL pattern: `https://officialskills.sh/<org>/skills/<skill>` (single `skills` segment).

**`clawhub.ai`** is a community marketplace referenced by Multica's docs but is small (~50 skills); none of this roster's canonical skills are mirrored there. Last-resort fallback only.
