# Skills + MCPs for Chief of Staff

## Skills (Multica → `agent_skill` table)

The Chief of Staff is light on attached skills, heavy on MCP queries and custom rubrics. The user's locally-installed `gsd-*`, `schedule`, `loop`, `init`, `find-skills` skills operate on the local Claude Code harness — they aren't portable into Multica's `agent_skill` table. The cadence + dispatch logic therefore lives in three places:

- the **system prompt** (decision rules — when to ask the human, what counts as "ready to assign")
- **Multica MCP queries** (issue/agent/autopilot data — they're queries, not skills)
- **custom skills authored via `skill-creator`** (the digest/triage/grooming patterns; see "To author" below)

### To attach (import from skills.sh)

| Skill | Why for this role | skills.sh | GitHub fallback |
|---|---|---|---|
| `internal-comms` | Status reports, newsletters, FAQs — the *voice* of every digest this agent ships. | [skills.sh/anthropics/skills/internal-comms](https://skills.sh/anthropics/skills/internal-comms) | [github.com/anthropics/skills/tree/main/internal-comms](https://github.com/anthropics/skills/tree/main/internal-comms) |
| `skill-creator` | When the agent identifies a gap (no agent matches an issue), it proposes authoring a new skill. | [skills.sh/anthropics/skills/skill-creator](https://skills.sh/anthropics/skills/skill-creator) | [github.com/anthropics/skills/tree/main/skill-creator](https://github.com/anthropics/skills/tree/main/skill-creator) |
| `mcp-builder` | Paired with the recommendation below — if no first-party Multica MCP exists, this agent proposes building one. | [skills.sh/anthropics/skills/mcp-builder](https://skills.sh/anthropics/skills/mcp-builder) | [github.com/anthropics/skills/tree/main/mcp-builder](https://github.com/anthropics/skills/tree/main/mcp-builder) |

### To author via `skill-creator`

| Skill (proposed) | Purpose |
|---|---|
| `agent-dispatch-rubric` | Codifies the routing decision tree: read attached skills → match to issue scope → fall back to "ask human". Removes hunch-routing. |
| `digest-template` | House-style daily/weekly/monthly digest scaffolds — KPI block, shipped block, blockers block, asks block. Keeps format stable across runs. |
| `triage-rubric` | P0–P3 priority + project assignment heuristics tuned to this workspace's taxonomy. |
| `inbound-capture` | When a human comment requests a future todo, file a Multica issue with project=`inbox`, label=`triage`, assignee=null. |
| `backlog-grooming-rubric` | Heuristics for the Friday grooming Autopilot: stale = no activity ≥14 days · dupe candidate = title cosine ≥ 0.85 · orphan = no project assignment · ready = has acceptance criteria + assignee-eligible. |
| `forward-looking-capture` | "Remember this for milestone X" ideas → Multica issue with a `trigger:milestone-X` label and target milestone. |
| `progress-snapshot` | The "what shipped this week" query template: Multica issues closed in last 7d × GitHub PRs merged × Sentry error-rate delta. Feeds `digest-template`. |
| `multica-cli-wrapper` *(only if Multica MCP path (a) below is chosen)* | Wraps `multica issue …`, `multica autopilot …`, `multica skill list` calls for the agent to use through Bash. |

## MCP servers (`agent.mcp_config`)

**Hard rule for this agent**: no code-edit, no DB-write, no deploy MCPs. Filesystem MCP is **not attached** — Chief of Staff has no business reading/writing repo files. GitHub is read-only. Slack and email are draft-and-queue only — paired with the system-prompt imperative.

| MCP | Scope | Why for this role |
|---|---|---|
| **Multica MCP** (community: `Korkyzer/multica-mcp`) — see notes below | write (issue/agent/autopilot CRUD) | The agent needs to read the workspace's issues, comments, agent roster, autopilot history, runtime utilization. Highest-leverage MCP for the role. |
| **GitHub MCP** (`github/github-mcp-server`) | **read-only** — issues, PRs, comments, status checks. *Disable* `create_or_update_file`, `push_files`, `merge_pull_request`, `delete_*`. | Cross-link Multica issues to GitHub PRs in the daily digest; surface PR status without touching code. |
| **Slack MCP** (`@modelcontextprotocol/server-slack`, GA Feb 2026) | **draft-only**: list channels/users + post gated through human approval queue. | Daily standup + weekly digest delivery. The agent never auto-posts; a human one-clicks send. |
| **Google Calendar MCP** (Google's official remote MCP, 2026) | **read-only** — list events, free/busy. | Deadline awareness in the weekly digest. Never schedules, never edits. |

### How to wire this in Multica

The cleaned, spec-valid config lives at `agents/chief-of-staff/mcp.json`. Multica's agent-settings GUI does not expose an MCP page today — [PR #1221](https://github.com/multica-ai/multica/pull/1221) is open. The daemon DOES read `agent.mcp_config` from the DB and pass it to Claude Code via `--mcp-config <tempfile>` (PR #1168, shipped v0.2.6). Full procedure in `agents/MCP-WIRING.md` Part A.

In the Multica GUI (Settings → Agents → Chief of Staff):
- **Custom arguments**: leave empty, or use for non-MCP tuning (e.g. `--model claude-opus-4-5`).
- **Environment** (one `KEY=VALUE` per line — referenced as `${VAR}` in `mcp.json`):
  ```
  MULTICA_PAT=mul_...
  GITHUB_PAT_CHIEF_OF_STAFF=ghp_...
  SLACK_BOT_TOKEN=xoxb-...
  SLACK_TEAM_ID=T...
  ```
  `google-calendar` is OAuth — completed once on first agent run via Claude Code's `/mcp` panel; no env var needed. Tool-scoping (read-only GitHub, draft-only Slack) is enforced by the **PAT scope itself** — see `agents/MCP-WIRING.md` Part C1 for the per-agent token-scope table.

**Skills bundled with Claude Code** (no marketplace import needed; auto-load when runtime is `claude-code`): none relevant for this role — Chief of Staff doesn't run `simplify`/`review`/`security-review` because it doesn't edit code.

### Multica MCP availability — two paths

There is **no first-party `multica-ai/multica-mcp`** as of May 2026. The community option is `Korkyzer/multica-mcp` (MIT, ~25 tools wrapping the `multica` CLI; tracked as feature request [multica-ai/multica#1351](https://github.com/multica-ai/multica/issues/1351)).

- **Path (a) — short-term**: attach `Korkyzer/multica-mcp` *or* author a workspace-local `multica-cli-wrapper` skill via `skill-creator` that shells out to the `multica` CLI through Bash. Lower trust surface; auditable in one file. Recommended until first-party support lands.
- **Path (b) — high-leverage**: this is the **#1 MCP to build first** with `mcp-builder`. The Chief of Staff benefits most from a stable, typed Multica API; every other agent benefits incidentally. File this as a Tier-1 self-improvement issue in week 2.

## Coherence check

Prompt + skills + MCPs all converge on one job: **read the workspace, decide who owns what, draft the comms, queue for human approval**. `internal-comms` covers voice. `skill-creator` + `mcp-builder` cover the agent's growth path — when it sees a gap, it can propose closing it instead of just complaining. The MCPs are deliberately read-heavy, with the only "write" being Multica issue/agent/autopilot CRUD (the agent's actual job) and Slack drafts (gated through approval). Code, deploy, and DB MCPs are explicitly absent — coded into the system prompt as imperatives.

### Sources
- Multica MCP feature request: https://github.com/multica-ai/multica/issues/1351
- Community Multica MCP: https://github.com/Korkyzer/multica-mcp
- GitHub MCP: https://github.com/github/github-mcp-server
- Slack MCP (official, GA Feb 2026): https://www.npmjs.com/package/@modelcontextprotocol/server-slack
- Google Calendar MCP: https://developers.google.com/workspace/calendar/api/guides/configure-mcp-server
- `skill-creator`, `mcp-builder`, `internal-comms`: https://github.com/anthropics/skills
