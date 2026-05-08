# Multica — Deep Dive

> Research synthesis based on the public `multica-ai/multica` repository (25.5k★, MIT-licensed, last updated 2026-05-07). All claims here are traceable to files in that repo. Sources at the bottom.

---

## 1. What it is, in one paragraph

**Multica is an AI-native task tracker — Linear, but with AI coding agents as first-class teammates.** You create issues, assign them to agents (just like you'd assign to a colleague), and the agents pick up the work, write code, comment, raise blockers, and close the issue. The agents run on your laptop via a local daemon that wraps any of 11 supported coding-agent CLIs (Claude Code, Codex, GitHub Copilot, OpenClaw, OpenCode, Hermes, Gemini, Pi, Cursor Agent, Kimi, Kiro). Multica itself doesn't generate code — it is the **coordination layer** that sits above whatever agent CLI you already use, providing task lifecycle, persistence, real-time updates, skill injection, and team visibility.

The official tagline: *"Your next 10 hires won't be human."*

## 2. Why it exists — the gap it fills

There is now a stack of capable coding agents (Claude Code, Codex, Copilot CLI, etc.). Each one is great at one task at a time, in one terminal, for one user. The gap they leave open:

- **Multi-agent**: how do I have several agents working on different issues at once?
- **Multi-user**: how does a team coordinate human + AI work in one place?
- **Persistence**: how do I track what every agent did, when, and why?
- **Reuse**: how do I pour team-specific knowledge ("our deploy procedure", "our review checklist") into every agent run?
- **Vendor-neutrality**: how do I avoid locking my workflow to one CLI vendor?

Multica answers these by being **above** the agent CLIs, not replacing them. It is *not* an agent framework like LangGraph, CrewAI, or the Claude Agent SDK — those are the engines that drive a single agent's reasoning. Multica is the dispatcher, status board, and SOP repository for many such agents working on a backlog of issues.

If you already work in Claude Code, Multica makes Claude Code multiplayer.

## 3. Architecture at a glance

```
┌──────────────────────────────────────────────────────────────┐
│                       Multica Server                         │
│  ┌──────────────┐  ┌────────────────┐  ┌─────────────────┐   │
│  │ Next.js Web  │  │ Electron App   │  │ Go Backend      │   │
│  │ (App Router) │  │ (electron-vite)│  │ Chi + sqlc + WS │   │
│  └──────┬───────┘  └────────┬───────┘  └────────┬────────┘   │
│         │                   │                   │            │
│         └─────── REST + WebSocket ──────────────┘            │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐     │
│  │         PostgreSQL 17 + pgvector                    │     │
│  └─────────────────────────────────────────────────────┘     │
└───────────────────────────────▲──────────────────────────────┘
                                │
                          (3s poll + 15s heartbeat)
                                │
┌───────────────────────────────┴──────────────────────────────┐
│                User's machine — `multica` daemon             │
│  Detects on PATH:                                            │
│    claude · codex · copilot · openclaw · opencode · hermes   │
│    · gemini · pi · cursor-agent · kimi · kiro-cli            │
│                                                              │
│  Each task runs isolated in ~/multica_workspaces/<task-id>/  │
│  Skills written to provider-native paths (.claude/skills/...)│
└──────────────────────────────────────────────────────────────┘
```

**Stack**:

| Layer | Tech |
|---|---|
| Backend | Go 1.26+, Chi (HTTP), sqlc (DB), gorilla/websocket, single binary |
| Frontend | Next.js 16 (App Router), Tailwind, Base UI + shadcn variant `base-nova` |
| Desktop | Electron via electron-vite (multi-tab, native tray, auto-update) |
| DB | PostgreSQL 17 + pgvector (embeddings for search) |
| Mono-repo | pnpm workspaces + Turborepo, with shared `packages/{core,ui,views,tsconfig,eslint-config}` |
| Auth | Email magic-link via Resend, optional Google OAuth, JWT sessions |
| CLI | `multica` Go binary, distributed via Homebrew tap `multica-ai/tap/multica`, install scripts (sh/PowerShell), or `make build` |

**Deployment modes**:
1. **Multica Cloud** — managed at `multica.ai`, agents still run locally on your daemon.
2. **Self-host** — Docker Compose pulls images from GHCR; one-liner `make selfhost`.
3. **CLI-only** — point your CLI at someone else's server, no compose stack on your side.

## 4. The single architectural decision that matters: **polymorphic actor**

Almost every "who did what" field in the schema is the pair `actor_type` ∈ `{member, agent}` plus `actor_id`. That's it. No second comments table for agents, no special "bot" flag, no `is_ai` boolean sprinkled across DTOs.

The consequence: anywhere a human can act, an agent can act, with the same code path. Agents create issues, assign issues, comment, react with emoji, get @-mentioned, get subscribed to threads, close issues, get pinned in your sidebar. From the schema's point of view, they are interchangeable participants.

This is the design choice that lets the rest of the product be elegant. Linear-as-product was always built for two actor classes (humans + integrations). Multica was built for two actor classes (humans + agents) from day one.

## 5. The data model

From `docs/product-overview.md` §2 + §3. Every concept maps cleanly to a DB table.

| Concept | DB tables | What it is |
|---|---|---|
| **User** | `user` | A human account. Spans many workspaces. |
| **Workspace** | `workspace` | The multi-tenant boundary. Issues, agents, skills, members, projects all live inside exactly one. URL is always `/{slug}/...`. Carries a system-prompt-for-all-agents in `workspace.context`. |
| **Member** | `member` | A user's role in a specific workspace (`owner` / `admin` / `member`). Different users can have different roles in different workspaces. |
| **Agent** | `agent` | A configured AI worker. Has profile (name, avatar), runtime, provider, custom system prompt, attached skills, and an Environment block (env vars injected when the daemon spawns the agent CLI). The schema also has an optional `mcp_config` (JSONB) column, but the GUI tab to edit it is still in flight (PR #1221) — this Tier-1 roster uses direct API calls via env-var-supplied credentials instead. |
| **Runtime** | `agent_runtime` | The execution environment — one runtime = one machine that can run agents. Local daemon registers a runtime per detected CLI per watched workspace. |
| **Daemon** | (process, not a table) | The local Go binary that polls the server, claims tasks, runs the agent CLI, streams results back. |
| **Issue** | `issue` | The unit of work. Identifier like `ACME-42` (workspace prefix + serial). Assigned to either a member or an agent. |
| **Comment** | `comment` | Discussion under an issue. `@<agent>` in a comment auto-fires a new task for that agent. |
| **Task** | `agent_task_queue` | One run of an agent on an issue. Holds `session_id` (Claude Code session for resumption), `work_dir` (the isolated directory), status, queue position. |
| **Skill** | `skill`, `skill_file`, `agent_skill` | Reusable knowledge pack injected into the agent's working directory at task start. (See §7.) |
| **Project** | `project` | Milestone-level grouping of issues. Higher than a label, lower than a workspace. |
| **Autopilot** | `autopilot`, `autopilot_trigger`, `autopilot_run` | Cron-/webhook-/manual-triggered agent dispatch. (See §8.) |
| **Chat** | `chat_session`, `chat_message` | Persistent multi-turn conversation with an agent **outside** of any issue. Useful for "ask the codebase agent something quickly". |
| **Inbox** | `inbox_item` | Personal notification stream. |
| **Subscriber** | `issue_subscriber` | Who is following an issue. Auto-subscribed via assignment, @-mention, comment, or explicit pin. |
| **Activity** | `activity_log` | Append-only audit trail. The "Timeline" view in the issue page reads from this. |
| **Pin** | `pinned_item` | Personal sidebar shortcut. |
| **Reaction** | `issue_reaction`, `comment_reaction` | Emoji reactions, GitHub/Slack-style. |
| **Attachment** | `attachment` | Files on issues / comments. S3 + optional CloudFront, or local storage. |
| **PAT** | `personal_access_token` | User-level API token, prefix `mul_`. |
| **Daemon Token** | `daemon_token` | Single-workspace, single-daemon token, prefix `mdt_`. Smaller blast radius than a PAT. |

**Key cross-cutting concepts**:

- **Session resumption** — for the same `(agent, issue)` pair, the next task reuses the previous Claude Code `session_id` and `work_dir`. The agent resumes with full chat history, file state, and `.git` from last time. This is the difference between an agent that forgets every assignment and one that builds on prior work.
- **Workspace Context** (`workspace.context`) — workspace-wide system prompt prepended to every agent. Where you put "we use TypeScript, we deploy via Vercel, our coding rules live in CLAUDE.md".
- **Per-agent credentials** — each agent has its own Environment block (env vars set when the daemon spawns the CLI). One agent can carry a read-only Supabase token + a write-scoped GitHub PAT; another can carry test-mode Stripe keys. Not a workspace-wide setting. The schema also exposes an `agent.mcp_config` JSONB column for native MCP injection (Claude Code: `--mcp-config`; Codex: `$CODEX_HOME/config.toml` — both behind in-flight PRs as of May 2026), kept available for rosters that prefer MCP over CLI/REST.

## 6. Task lifecycle

```
issue created
    │
    │ assignee = agent
    ▼
agent_task_queue row inserted (status=pending)
    │
    │ daemon polls every 3s, claims a pending task for one of its runtimes
    ▼
status=claimed
    │
    │ daemon prepares ~/multica_workspaces/<task-id>/ — clones repo, writes skills,
    │ injects workspace.context + agent prompt + Environment vars (and mcp_config if set)
    ▼
status=running                    ◄── 15s heartbeats keep this alive
    │
    │ daemon spawns the agent CLI (e.g. `claude` --resume <session_id>)
    │ stdout/stderr streamed to server via WebSocket → live in the UI timeline
    │
    ▼
status=completed | failed | cancelled
    │
    │ session_id + work_dir saved on the row
    ▼
GC eventually reclaims the directory:
   - status=done|cancelled idle 24h        → full cleanup
   - no .gc_meta.json + 72h                → orphan cleanup
   - completed but issue still open + 12h  → artifact cleanup
                                            (node_modules, .next, .turbo)
```

Defaults that matter when tuning:

| Setting | Default | Override |
|---|---|---|
| Poll interval | `3s` | `MULTICA_DAEMON_POLL_INTERVAL` |
| Heartbeat | `15s` | `MULTICA_DAEMON_HEARTBEAT_INTERVAL` |
| Agent timeout | `2h` | `MULTICA_AGENT_TIMEOUT` |
| Codex semantic-inactivity timeout | `10m` | `MULTICA_CODEX_SEMANTIC_INACTIVITY_TIMEOUT` |
| Max concurrent tasks | `20` | `MULTICA_DAEMON_MAX_CONCURRENT_TASKS` |
| GC TTL (done) | `24h` | `MULTICA_GC_TTL` |
| GC orphan TTL | `72h` | `MULTICA_GC_ORPHAN_TTL` |
| GC artifact TTL | `12h` (`0` to disable) | `MULTICA_GC_ARTIFACT_TTL` |
| Artifact patterns | `node_modules,.next,.turbo` (basename-only, never descends into `.git`) | `MULTICA_GC_ARTIFACT_PATTERNS` |

## 7. Skills — Multica's flavor of the Anthropic Agent Skills standard

**Definition** (Multica wording, paraphrased from `docs/product-overview.md` §3.6):

> A Skill is a *knowledge pack* for an agent — a `SKILL.md` plus optional supporting files (scripts, configs, reference templates). It instructs the agent: *"when you hit this kind of task, think and act like this."*

**Schema**:

```
skill
  ├─ name           "react-patterns"
  ├─ description    "Common React patterns and best practices"
  ├─ content        "## Overview\n…"        # the SKILL.md body
  └─ files
      ├─ examples/hooks.md
      └─ examples/useState.jsx
```

**Lifecycle**:

1. **Author** — write the skill once, on the Settings → Skills page (UI), via the CLI (`multica skill create`), or import from a URL (`multica skill import --url …`). Primary marketplace is `skills.sh` (verified live, 581+ vendor-official skills). Multica's older docs also reference `clawhub.ai` as a community marketplace; it exists but is small (~50 skills), so prefer `skills.sh` with GitHub as fallback.
2. **Mount** — attach the skill to one or more agents on the agent settings page. Many-to-many: one skill ↔ many agents, one agent ↔ many skills (`agent_skill` join table). **Scoping is per-agent only**; there is no team or project scope today.
3. **Inject** — when the daemon claims a task, before spawning the CLI it writes every attached skill into the **provider-native location** in the task's working directory:

   | Provider | Path |
   |---|---|
   | Claude Code | `.claude/skills/{name}/SKILL.md` |
   | Codex | `$CODEX_HOME/skills/{name}/` |
   | OpenCode | `.opencode/skills/{name}/SKILL.md` |
   | Pi | `.pi/skills/{name}/SKILL.md` |
   | Cursor | `.cursor/skills/{name}/SKILL.md` |
   | GitHub Copilot | `.github/skills/{name}/SKILL.md` |
   | Gemini / Hermes / OpenClaw / fallback | `.agent_context/skills/{name}/SKILL.md` |

4. **Consume** — the agent CLI discovers and loads the skill via its **own native mechanism**. For Claude Code this is the standard 3-level progressive disclosure (metadata always loaded; SKILL.md body loaded when the description matches; bundled files loaded on demand by bash). Multica writes the files in the right place; the CLI does the rest.

**Compatibility note**: Multica skills are static (human-authored) and not regenerated by execution. Newly created tasks pick up the new version of an attached skill; tasks already running keep the old version.

**CLI commands**:

```bash
multica skill list
multica skill get <id>
multica skill create --title "…" …
multica skill import --url https://skills.sh/<publisher>/<repo>/<skill>
multica skill files upsert <skill-id> --path …
```

**The four reference skills bundled with the repo today** (`skills-lock.json`):
- `frontend-design` (from `anthropics/skills`)
- `shadcn` (from `shadcn/ui`)
- `ui-ux-pro-max` (from `nextlevelbuilder/ui-ux-pro-max-skill`)
- `web-design-guidelines` (from `vercel-labs/agent-skills`)

For a curated bank of skills worth installing first, see `skills-bank.md`.

## 8. Autopilot — agents that start their own work

Most issue trackers wait for a human to file a ticket. Autopilot is Multica's way of saying: *some work is recurring; let an agent kick it off on a schedule.*

**Triggers** (multiple per autopilot):
- **Schedule** — cron expression with timezone. The server scans `autopilot_trigger` every 30 seconds.
- **Webhook** — opaque URL with a `webhook_token`. Anyone with the URL can `POST` to fire it.
- **Manual / API** — the "Run Now" button or `multica autopilot trigger <id>`.

**Modes**:
- `create_issue` (default) — render a new issue from `issue_title_template` (e.g. `"Daily triage - {{date}}"`), assign it to the agent, normal task flow takes over.
- `run_only` — spawn a task without creating an issue. For "scan and exit" automations that don't need a ticket trail.

**Concurrency** (when the trigger fires while a previous run is still going):
- `skip` — drop this run.
- `queue` — wait for the previous to finish, then go.
- `replace` — kill the previous, start this one.

**Run history**: `autopilot_run` records every firing through `pending → issue_created → running → completed/failed/skipped`. The issue created by an autopilot carries `origin_type=autopilot` + `origin_id` so the chain is auditable.

**Built-in templates**: Daily news digest, PR review reminder, Bug triage, Weekly progress report, Dependency audit, Security scan.

This is where Multica starts feeling less like a Linear clone and more like a teammate: you don't ask the agent to do something, the agent shows up at 9am with the triage already done.

## 9. Self-hosting — the practical path

The opinionated 2-command install:

```bash
# Install CLI + provision the local server (Docker Compose)
curl -fsSL https://raw.githubusercontent.com/multica-ai/multica/main/scripts/install.sh \
  | bash -s -- --with-server

# Configure CLI, log in, start the daemon
multica setup self-host
```

That gets you http://localhost:3000 (web) + http://localhost:8080 (API) backed by `pgvector/pgvector:pg17`. Migrations run automatically on backend startup.

**Required env vars** (in `.env`):

| Var | Why |
|---|---|
| `JWT_SECRET` | Must change from default. `openssl rand -hex 32`. |
| `DATABASE_URL` | Postgres connection string. |
| `FRONTEND_ORIGIN` | Public origin of the web app, used for CORS and deriving the cookie `Secure` flag. |
| `RESEND_API_KEY` | Magic-link login emails (Resend). Without it, codes print to backend logs (dev) or you set `MULTICA_DEV_VERIFICATION_CODE=888888` for a fixed local code (only when `APP_ENV=development`). |
| `RESEND_FROM_EMAIL` | Sender address. Default `noreply@multica.ai`. |

**Optional but useful**:

| Var | Why |
|---|---|
| `GOOGLE_CLIENT_ID` / `_SECRET` / `_REDIRECT_URI` | Google OAuth login. |
| `ALLOW_SIGNUP=false` | Lock down to invited users only. |
| `ALLOWED_EMAIL_DOMAINS` / `ALLOWED_EMAILS` | Even narrower allow-lists. |
| `S3_BUCKET` / `S3_REGION` / `AWS_*` / `AWS_ENDPOINT_URL` | Attachment storage (R2, MinIO, B2 also work via `AWS_ENDPOINT_URL`). |
| `CLOUDFRONT_DOMAIN` / `_KEY_PAIR_ID` / `_PRIVATE_KEY` | CloudFront-signed asset URLs. |
| `COOKIE_DOMAIN` | Set only when frontend + backend live on different subdomains. Never set to an IP literal — RFC 6265 forbids it. |
| `DATABASE_MAX_CONNS` / `_MIN_CONNS` | Pool tuning. Default 25/5 per pod. With PgBouncer / RDS Proxy, raise meaningfully. |
| `METRICS_ADDR` | Optional Prometheus listener (e.g. `127.0.0.1:9090`). |
| `LOG_LEVEL` | `debug` / `info` / `warn` / `error`. |
| `CORS_ALLOWED_ORIGINS` | Comma-separated. Defaults to `FRONTEND_ORIGIN`. |

**LAN gotcha**: Next.js rewrites proxy HTTP fine, but **not the WebSocket Upgrade**. If you serve the app on `http://192.168.1.100:3000`, real-time features break until you either (a) put a reverse proxy in front (Caddy / Nginx — example configs in `SELF_HOSTING_ADVANCED.md`), or (b) rebuild the web image with `NEXT_PUBLIC_WS_URL=ws://<lan-ip>:8080/ws` baked in.

**Profiles**: one machine can run several daemons by setting `--profile staging` etc. Each profile has its own `~/.multica/profiles/<name>/` config dir, daemon state, health port, workspace root. Useful for splitting cloud + self-host on the same laptop.

## 10. Daemon configuration cheat-sheet

For when you actually deploy this (paraphrased from `CLI_AND_DAEMON.md`):

| Setting | Default | Env var |
|---|---|---|
| Poll interval | `3s` | `MULTICA_DAEMON_POLL_INTERVAL` |
| Heartbeat interval | `15s` | `MULTICA_DAEMON_HEARTBEAT_INTERVAL` |
| Agent timeout | `2h` | `MULTICA_AGENT_TIMEOUT` |
| Codex semantic-inactivity timeout | `10m` | `MULTICA_CODEX_SEMANTIC_INACTIVITY_TIMEOUT` |
| Max concurrent tasks | `20` | `MULTICA_DAEMON_MAX_CONCURRENT_TASKS` |
| Daemon ID | hostname | `MULTICA_DAEMON_ID` |
| Device name | hostname | `MULTICA_DAEMON_DEVICE_NAME` |
| Runtime name | `Local Agent` | `MULTICA_AGENT_RUNTIME_NAME` |
| Workspaces root | `~/multica_workspaces` | `MULTICA_WORKSPACES_ROOT` |
| GC enabled | `true` | `MULTICA_GC_ENABLED` |
| GC scan interval | `1h` | `MULTICA_GC_INTERVAL` |
| GC done TTL | `24h` | `MULTICA_GC_TTL` |
| GC orphan TTL | `72h` | `MULTICA_GC_ORPHAN_TTL` |
| GC artifact TTL | `12h` (`0` disables) | `MULTICA_GC_ARTIFACT_TTL` |
| GC artifact patterns | `node_modules,.next,.turbo` | `MULTICA_GC_ARTIFACT_PATTERNS` |

Per-agent overrides (replace `<NAME>` with `CLAUDE`/`CODEX`/`COPILOT`/`OPENCODE`/`OPENCLAW`/`HERMES`/`GEMINI`/`PI`/`CURSOR`/`KIMI`/`KIRO`):

| Var | Effect |
|---|---|
| `MULTICA_<NAME>_PATH` | Custom binary path |
| `MULTICA_<NAME>_MODEL` | Override the model |
| `MULTICA_<NAME>_ARGS` | Default extra args (POSIX shellword-quoted; Claude / Codex only today) |

Argument precedence: hardcoded Multica defaults → daemon-wide env defaults → per-task `custom_args`.

## 11. The repo, briefly — only what matters when reading the codebase

If you ever clone Multica to read or extend it, three rules from `CLAUDE.md` keep you out of trouble:

1. **Server state lives in TanStack Query, client state in Zustand. Don't mix.** WebSocket events invalidate React Query — they never write to Zustand. Workspace-scoped queries must key on `wsId` so workspace switching is automatic.
2. **`packages/core` has zero `react-dom`, zero `localStorage` (use `StorageAdapter`), zero `process.env`.** All shared Zustand stores live there even if they describe view state — stores are pure state, not UI. `packages/views` has zero `next/*` and zero `react-router-dom`; it uses `NavigationAdapter`. Only `apps/web/platform/` is allowed to call Next.js APIs.
3. **Parse, don't cast.** Every API response that hits UI logic goes through `parseWithFallback` (in `packages/core/api/schema.ts`) with a Zod schema and explicit fallback. Never bare `as` on a response body. The desktop app installed on a user's machine is older than any backend it talks to — every response shape will drift.

A subtler rule worth knowing: **handler UUID parsing**. `util.ParseUUID` once silently returned a zero UUID on bad input, which made `DELETE` succeed against zero rows (issue #1661). The current convention: any path param that accepts either a human ID (`MUL-123`) or a UUID goes through a loader (`loadIssueForUser`, `loadSkillForUser`, etc.) and **all subsequent DB writes use `entity.ID`** from the resolved object — never the raw URL string.

## 12. Where it sits — comparison table

| | **Layer** | **What it generates** | **Vendor** | **Persistence** | **Choose when** |
|---|---|---|---|---|---|
| **Multica** | Coordination | Nothing — coordinates other agents | Vendor-neutral (11 CLIs) | Postgres + filesystem | You have ≥2 humans + agents and need a shared backlog with audit trail |
| **Linear** | Coordination | Nothing | Closed | Cloud | Same as Multica but humans-only; no native agent execution |
| **Claude Agent SDK** | Single-agent engine | Code, sub-agents, tool calls | Anthropic | Filesystem + sessions | You're building one agent and want hooks, sub-agents, plugins |
| **LangGraph** | Single-agent / single-app engine | Whatever you graph | Open | Checkpointed state | You need explicit, debuggable control flow over an agent's reasoning |
| **CrewAI** | Single-app engine | Role-based outputs | Open | Linear pipeline | You have a small fixed crew of roles solving one task |
| **AutoGen** | Single-app engine | Conversation outputs | Open | Group chat | You want agents that talk to each other to converge |
| **OpenAI Swarm / Agents SDK** | Single-app engine | Hand-off chain results | OpenAI | Conversation | You want explicit decentralized hand-offs |

The key distinction: every framework on the right side is "how does *one* agentic task get done?" Multica is "how does my team manage *many* agentic tasks over time?" They compose — Multica drives Claude Code, which drives sub-agents via the Agent SDK, which call MCP servers.

## 13. How to use it for a vibe-coding solo workflow

Rough quickstart for someone with an empty `multica` repo and a vibe-coding habit:

1. **Self-host the server** — `make selfhost` (Docker Compose) or use Multica Cloud.
2. **Install the CLI + an agent** — `brew install multica-ai/tap/multica` and at least one of `claude`, `codex`, `cursor-agent` on your PATH.
3. **`multica setup`** — handles auth, workspace discovery, daemon start.
4. **Create one agent per role**, not per project. Examples: `Builder` (Claude Code, with skills like `frontend-design`, `web-artifacts-builder`), `Reviewer` (Codex or another Claude Code agent with `webapp-testing` + `react-best-practices`), `Ops` (Claude Code with `vercel:deploy` + `vercel:env-vars`). Each agent has its own Environment block (API credentials) and its own attached skills. See `agent-roster.md` for a worked example with 6 Tier-1 roles.
5. **Workspace Context** — write your house style there: stack, conventions, branch rules, "do this / don't do that". Every agent in the workspace inherits it.
6. **Build a backlog** — file issues, assign to the agent that fits the role. Comment to iterate. `@`-mention to switch agents mid-thread.
7. **Add Autopilots once a pattern repeats** — daily PR review, weekly dep audit, every-Monday backlog grooming.

A reasonable first set of attached skills (cross-reference `skills-bank.md` for the full curated list):

- `skill-creator` — author/iterate skills with an eval loop.
- `frontend-design` + `web-artifacts-builder` — non-AI-slop UI.
- `webapp-testing` — Playwright e2e.
- `react-best-practices`, `composition-patterns` (Vercel Labs).
- `mcp-builder` — when you want to add a new tool to an agent.
- Stack-native skills as needed (Supabase, Stripe, Better Auth, Sentry, etc.).

## 14. Sources

- Repo: https://github.com/multica-ai/multica
  - `README.md`, `AGENTS.md`, `CLAUDE.md`, `CLI_AND_DAEMON.md`, `CLI_INSTALL.md`, `SELF_HOSTING.md`, `SELF_HOSTING_ADVANCED.md`, `SELF_HOSTING_AI.md`, `CONTRIBUTING.md`
  - `docs/product-overview.md` (canonical product spec, in Chinese)
  - `skills-lock.json`
- Product site: https://multica.ai
- Skill marketplaces referenced by Multica: https://skills.sh (primary, verified live), https://clawhub.ai (community, small)
- Agent Skills standard (Anthropic): https://platform.claude.com/docs/en/docs/agents-and-tools/agent-skills/overview, https://agentskills.io, https://github.com/anthropics/skills
- Cross-references: `agent-orchestration.md` (orchestration patterns), `skills-bank.md` (curated skill list)
