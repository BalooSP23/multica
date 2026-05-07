# Skills + MCPs for Chief of Staff

## Skills (Multica → `agent_skill` table)

### Already installed locally (just attach)
**None directly applicable.** The user's locally-installed `gsd-*`, `schedule`, `loop`, `init`, `find-skills`, `update-config`, and `keybindings-help` skills operate on the **local Claude Code harness** (the user's terminal session) — they aren't portable to Multica's per-agent `agent_skill` table. Multica injects skills from its own `skill` table into a task's `work_dir`; nothing crosses over from `~/.claude/skills`.

The Chief of Staff's cadence + dispatch logic therefore lives in three places, none of which is a pre-existing local skill:
- the **system prompt** (decision rules — when to ask the human, what counts as "ready to assign")
- the **Multica MCP** (where "what shipped this week", "stale issues", "agent utilization" actually come from — they're queries, not skills)
- the **custom skills authored via `skill-creator`** (see "To author" below — these encode the digest/triage/grooming patterns the local `gsd-*` skills cover for your terminal)

### Needs installation
| Skill | Source | skills.sh | clawhub.io | Why for this role |
|---|---|---|---|---|
| `internal-comms` | `anthropics/skills/internal-comms` | [skills.sh/anthropics/skills/internal-comms](https://skills.sh/anthropics/skills/internal-comms) | unreachable (cert/domain expired May 2026) — fallback: [github.com/anthropics/skills/tree/main/internal-comms](https://github.com/anthropics/skills/tree/main/internal-comms) | Status reports, newsletters, FAQs — the *voice* of every digest this agent ships. Cited in `skills-bank.md` §3 / Tier 3. |
| `skill-creator` | `anthropics/skills/skill-creator` | [skills.sh/anthropics/skills/skill-creator](https://skills.sh/anthropics/skills/skill-creator) | unreachable — fallback: [github.com/anthropics/skills/tree/main/skill-creator](https://github.com/anthropics/skills/tree/main/skill-creator) | When the agent identifies a gap (no agent matches an issue), it proposes authoring a new skill. `skills-bank.md` §3, Tier 1 essential. |
| `mcp-builder` | `anthropics/skills/mcp-builder` | [skills.sh/anthropics/skills/mcp-builder](https://skills.sh/anthropics/skills/mcp-builder) | unreachable — fallback: [github.com/anthropics/skills/tree/main/mcp-builder](https://github.com/anthropics/skills/tree/main/mcp-builder) | Paired with the recommendation below — if no official Multica MCP, this agent proposes building one. `skills-bank.md` §3. |

### To author via `skill-creator`
| Skill (proposed) | Purpose |
|---|---|
| `agent-dispatch-rubric` | Codifies the routing decision tree: read attached skills → match to issue scope → fall back to "ask human". Removes hunch-routing. |
| `digest-template` | House-style daily/weekly/monthly digest scaffolds — KPI block, shipped block, blockers block, asks block. Keeps the format stable across runs. |
| `triage-rubric` | P0–P3 priority + project assignment heuristics tuned to this workspace's taxonomy. |
| `inbound-capture` | When a human comment requests a future todo, file a Multica issue with project=`inbox`, label=`triage`, assignee=null. Multica-native equivalent of the local `gsd-add-todo` flow. |
| `backlog-grooming-rubric` | Heuristics for the Friday grooming Autopilot: stale = no activity ≥14 days · dupe candidate = title cosine ≥ 0.85 · orphan = no project assignment · ready = has acceptance criteria + assignee-eligible. Multica-native equivalent of `gsd-review-backlog` + `gsd-check-todos`. |
| `forward-looking-capture` | "Remember this for milestone X" ideas → Multica issue with a `trigger:milestone-X` label and target milestone. Multica-native equivalent of `gsd-plant-seed`. |
| `progress-snapshot` | The "what shipped this week" query template: Multica issues closed in last 7d × GitHub PRs merged × Sentry error-rate delta. The output that feeds `digest-template`. Equivalent of `gsd-progress` + `gsd-stats`, but pulls from Multica/GitHub MCPs instead of `.planning/`. |
| `multica-cli-wrapper` *(only if Multica MCP path (a) below is chosen)* | Wraps `multica issue …`, `multica autopilot …`, `multica skill list` calls for the agent to use through Bash. |

## MCP servers (`agent.mcp_config`)

**Hard rule for this agent**: no code-edit, no DB-write, no deploy MCPs. Filesystem MCP is **not attached** — Chief of Staff has no business reading/writing repo files. GitHub is read-only (issues/PRs only, no `create_or_update_file`, no `push_files`, no `merge_pull_request`). Slack and email are draft-and-queue only — paired with the system-prompt imperative.

| MCP | Scope | Why for this role |
|---|---|---|
| **Multica MCP** (community: `Korkyzer/multica-mcp`) — see notes below | write (issue/agent/autopilot CRUD) | The agent needs to read its own workspace's issues, comments, agent roster, autopilot history, and runtime utilization. This is the highest-leverage MCP for the role. |
| **GitHub MCP** (`github/github-mcp-server`, formerly `@modelcontextprotocol/server-github`) | **read-only** — issues, PRs, comments, status checks. *Disable* `create_or_update_file`, `push_files`, `merge_pull_request`, `delete_*`. | Cross-link Multica issues to GitHub PRs in the daily digest; surface PR status (open / awaiting review / merged) without touching code. |
| **Slack MCP** (`@modelcontextprotocol/server-slack` — official, GA Feb 2026) | **draft-only**: channel/user list + message *post* gated through human approval queue (the *agent* drafts; an Autopilot or human triggers the actual `chat.postMessage`). | Daily standup + weekly digest delivery. The agent never auto-posts; it generates the message body, an inbox item links it, and the human one-clicks send. |
| **Google Calendar MCP** (Google's official `calendarmcp.googleapis.com` remote MCP, announced 2026) | **read-only** — list events, free/busy. | Deadline awareness for the weekly digest ("3 deliverables due before Friday's investor call"). Never schedules, never edits. |

### How to wire this in Multica (no MCP UI page yet)

The cleaned, spec-valid config lives at `agents/chief-of-staff/mcp.json` (sibling file). Multica's agent-settings GUI does not expose an MCP page today — [PR #1221](https://github.com/multica-ai/multica/pull/1221) is still open as of May 2026. The daemon DOES however read `agent.mcp_config` from the DB and pass it to Claude Code via `--mcp-config <tempfile>` (PR #1168, shipped v0.2.6). So:

1. **Load `mcp.json` into the agent's `mcp_config` column** — via REST API, `multica` CLI, or direct SQL. Recipes in `agents/MCP-WIRING.md` Part A2. Custom arguments cannot inject `--mcp-config` (the daemon blocks it via `claudeBlockedArgs`).

2. **In the Multica GUI** (Settings → Agents → Chief of Staff):
   - **Custom arguments**: leave empty, or use for non-MCP tuning (e.g. `--model claude-opus-4-5`).
   - **Environment** (one `KEY=VALUE` per line — secrets used as `${VAR}` in `mcp.json`):
     ```
     MULTICA_PAT=mul_...
     GITHUB_PAT_CHIEF_OF_STAFF=ghp_...
     SLACK_BOT_TOKEN=xoxb-...
     SLACK_TEAM_ID=T...
     ```
     `google-calendar` is OAuth — completed once on first agent run via Claude Code's `/mcp` panel; no env var needed. Tool-scoping (read-only GitHub, draft-only Slack) is enforced by the **PAT scope itself**, not by JSON keys — see `agents/MCP-WIRING.md` Part C1 for the per-agent token-scope table.

Rationale notes that previously lived in `_comment`/`_allowed_tools` pseudo-fields:
- **multica**: community MCP `Korkyzer/multica-mcp`. See "Multica MCP availability" note below.
- **github**: PAT scoped to `issues:read`, `pull_requests:read`, `metadata:read` — write tools (`create_or_update_file`, `push_files`, `merge_pull_request`) are unreachable because the token cannot perform them.
- **slack**: token granted only `channels:read`, `users:read`, `channels:history` — `chat:write` deliberately absent so the agent literally cannot post.
- **google-calendar**: OAuth scope limited to `calendar.events.readonly` + `calendar.freebusy`.

### Multica MCP availability — two paths

There is **no first-party `multica-ai/multica-mcp`** as of May 2026. The community option is `Korkyzer/multica-mcp` (MIT, 4★, ~25 tools wrapping the `multica` CLI; tracked as feature request [multica-ai/multica#1351](https://github.com/multica-ai/multica/issues/1351)).

- **Path (a) — short-term**: attach `Korkyzer/multica-mcp` *or* author a workspace-local `multica-cli-wrapper` skill via `skill-creator` that shells out to the `multica` CLI through Bash. Lower trust surface than a third-party MCP; auditable in one file. Recommended until first-party support lands.
- **Path (b) — high-leverage**: this is the **#1 MCP to build first** with `mcp-builder`. The Chief of Staff is the agent that benefits most from a stable, typed Multica API; every other agent benefits incidentally. File this as a Tier-1 self-improvement issue in week 2.

## Coherence check
Prompt + skills + MCPs all converge on one job: **read the workspace, decide who owns what, draft the comms, queue for human approval**. `internal-comms` covers voice. `skill-creator` + `mcp-builder` cover the agent's growth path — when it sees a gap, it can propose closing it instead of just complaining. The cadence work (progress snapshots, todo capture, grooming, forward-looking parking) lives in the **custom skills authored via `skill-creator`** plus **Multica MCP queries** — not in any pre-existing local skill, because your local `gsd-*` set is bound to the Claude Code harness on your laptop and doesn't ride into a Multica agent's task. The MCPs are deliberately read-heavy, with the only "write" being Multica issue/agent/autopilot CRUD (the agent's actual job) and Slack drafts (gated through approval). Code, deploy, and DB MCPs are explicitly absent — coded into the system prompt as imperatives so even if a future operator adds them by mistake, the agent refuses. Leverage compounds because every other Tier-1 agent reads the digests and routing this agent produces.

### Sources
- Multica MCP feature request: https://github.com/multica-ai/multica/issues/1351
- Community Multica MCP: https://github.com/Korkyzer/multica-mcp
- GitHub MCP (current home): https://github.com/github/github-mcp-server (npm legacy: https://www.npmjs.com/package/@modelcontextprotocol/server-github)
- Slack MCP (official, GA Feb 2026): https://www.npmjs.com/package/@modelcontextprotocol/server-slack · https://docs.slack.dev/ai/slack-mcp-server/
- Google Calendar MCP (Google official remote MCP): https://developers.google.com/workspace/calendar/api/guides/configure-mcp-server
- `skill-creator`, `mcp-builder`, `internal-comms`: https://github.com/anthropics/skills
