# Skills + tools for Chief of Staff

## Skills (Multica → `agent_skill` table)

The Chief of Staff is light on attached skills, heavy on API queries and custom rubrics. The user's locally-installed `gsd-*`, `schedule`, `loop`, `init`, `find-skills` skills operate on the local Claude Code harness — they aren't portable into Multica's `agent_skill` table. The cadence + dispatch logic therefore lives in three places:

- the **system prompt** (decision rules — when to ask the human, what counts as "ready to assign")
- **API queries** (Multica REST API + GitHub API for issue/agent/autopilot/PR data)
- **custom skills authored via `skill-creator`** (digest/triage/grooming patterns; see "To author" below)

### To attach (import from skills.sh)

| Skill | Why for this role | skills.sh | GitHub fallback |
|---|---|---|---|
| `internal-comms` | Status reports, newsletters, FAQs — the *voice* of every digest this agent ships. | [skills.sh/anthropics/skills/internal-comms](https://skills.sh/anthropics/skills/internal-comms) | [github.com/anthropics/skills/tree/main/internal-comms](https://github.com/anthropics/skills/tree/main/internal-comms) |
| `skill-creator` | When the agent identifies a gap (no agent matches an issue), it proposes authoring a new skill. | [skills.sh/anthropics/skills/skill-creator](https://skills.sh/anthropics/skills/skill-creator) | [github.com/anthropics/skills/tree/main/skill-creator](https://github.com/anthropics/skills/tree/main/skill-creator) |

### To author via `skill-creator`

| Skill (proposed) | Purpose |
|---|---|
| `agent-dispatch-rubric` | Codifies the routing decision tree: read attached skills → match to issue scope → fall back to "ask human". Removes hunch-routing. |
| `digest-template` | House-style daily/weekly/monthly digest scaffolds — KPI block, shipped block, blockers block, asks block. Keeps format stable across runs. |
| `triage-rubric` | P0–P3 priority + project assignment heuristics tuned to this workspace's taxonomy. |
| `inbound-capture` | When a human comment requests a future todo, file a Multica issue with project=`inbox`, label=`triage`, assignee=null. |
| `backlog-grooming-rubric` | Heuristics for the Friday grooming Autopilot: stale = no activity ≥14 days · dupe candidate = title cosine ≥ 0.85 · orphan = no project assignment · ready = has acceptance criteria + assignee-eligible. |
| `forward-looking-capture` | "Remember this for milestone X" ideas → Multica issue with a `trigger:milestone-X` label and target milestone. |
| `progress-snapshot` | The "what shipped this week" query template: `multica issue list --status closed --since 7d` × `gh pr list --state merged --search merged:>7d` × Sentry `events.json` delta. Feeds `digest-template`. |
| `multica-cli-wrapper` | Wraps `multica issue …`, `multica autopilot …`, `multica skill list` calls so the agent can use them through Bash with structured output. |

## Tools & API access

**Hard rule for this agent**: read-heavy, write-only-via-draft. No code edits, no DB writes, no deploy commands. The agent reaches every external system through CLIs or HTTP APIs invoked from Bash — never as a server-side merge or auto-publish action.

| Service | How the agent uses it | Required env vars (set in Multica → Agent → Environment) | Scope |
|---|---|---|---|
| **Multica** | `multica` CLI for issue/agent/autopilot CRUD; or REST API via `curl $MULTICA_BASE/api/v1/...` with `Authorization: Bearer $MULTICA_PAT`. | `MULTICA_PAT` (`mul_…`) · `MULTICA_BASE` (e.g. `http://localhost:8080`) | Full workspace scope. The agent's actual job lives here. |
| **GitHub** | `gh` CLI for `gh issue list`, `gh pr list`, `gh pr view`, `gh issue comment`. Cross-link Multica issues to GitHub PRs in digests. | `GH_TOKEN=$GITHUB_PAT_CHIEF_OF_STAFF` (read scope only: `issues:read`, `pull_requests:read`, `metadata:read`) | **Read-only.** PAT lacks `contents:write`, `merge`, `workflows`. |
| **Slack** | `curl https://slack.com/api/chat.postMessage` with `Authorization: Bearer $SLACK_BOT_TOKEN`. Drafts go to a holding channel; humans forward to the team channel. | `SLACK_BOT_TOKEN` (`xoxb-…` with `chat:write` only on the `#draft-digests` channel) · `SLACK_TEAM_ID` | **Draft-only.** Token grants posting to one channel. |
| **Google Calendar** | Google Calendar REST API: `GET https://www.googleapis.com/calendar/v3/freeBusy`. OAuth-completed once at agent setup; refresh token lives in env. | `GCAL_REFRESH_TOKEN` (calendar.readonly + calendar.freebusy scopes) | Read-only. |

### Wiring in Multica

In the Multica GUI (Settings → Agents → Chief of Staff), set:

- **Custom arguments**: leave empty, or add `--model claude-opus-4-5` for digest synthesis.
- **Environment** (one `KEY=VALUE` per line):
  ```
  MULTICA_PAT=mul_...
  MULTICA_BASE=http://localhost:8080
  GH_TOKEN=ghp_...
  SLACK_BOT_TOKEN=xoxb-...
  SLACK_TEAM_ID=T...
  GCAL_REFRESH_TOKEN=1//0...
  ```

The daemon writes these as env vars when spawning the agent CLI; the agent invokes `gh`, `multica`, `curl` from Bash. No MCP server config to maintain.

**Skills bundled with Claude Code** (auto-load when runtime is `claude-code`): none relevant for this role — Chief of Staff doesn't run `simplify`/`review`/`security-review` because it doesn't edit code.

## Coherence check

Prompt + skills + API access converge on one job: **read the workspace, decide who owns what, draft the comms, queue for human approval**. `internal-comms` covers voice. `skill-creator` covers the agent's growth path. The API set is deliberately read-heavy, with the only "write" being Multica issue/agent/autopilot CRUD (the agent's actual job) and Slack drafts (gated through approval). Code, deploy, and DB API access is explicitly absent from the env-var set — coded into the system prompt as imperatives.

### Sources
- Multica REST API + CLI reference: https://github.com/multica-ai/multica/blob/main/CLI_AND_DAEMON.md
- GitHub CLI: https://cli.github.com/manual/
- Slack Web API (`chat.postMessage`): https://api.slack.com/methods/chat.postMessage
- Google Calendar API: https://developers.google.com/workspace/calendar/api/v3/reference/freebusy
- `skill-creator`, `internal-comms`: https://github.com/anthropics/skills
