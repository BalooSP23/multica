# Skills + MCPs for Backend Builder

## Skills

### Already installed locally (just attach)

> Note on URL columns: the user's "local" install only matters for their laptop's Claude Code harness. Multica's per-agent `agent_skill` table is separate — every skill below still needs `multica skill import --url <URL>` to be attached. `simplify`, `review`, `security-review`, `init` are bundled with Claude Code itself; they auto-load when the agent's runtime is `claude-code` and need no marketplace import.

| Skill | Source | skills.sh | clawhub.io | Why for this role |
|---|---|---|---|---|
| `claude-api` | `anthropics/skills/claude-api` | [skills.sh/anthropics/skills/claude-api](https://skills.sh/anthropics/skills/claude-api) | unreachable (cert/domain expired May 2026) — fallback: [github.com/anthropics/skills/tree/main/claude-api](https://github.com/anthropics/skills/tree/main/claude-api) | Backend touches the Anthropic SDK any time the product calls a model server-side; canonical reference for prompt caching, tool use, model migrations. |
| `supabase:supabase` | `supabase/agent-skills` (meta-router) | [skills.sh/supabase/agent-skills/supabase](https://skills.sh/supabase/agent-skills/supabase) | unreachable — fallback: [github.com/supabase/agent-skills/tree/main/supabase](https://github.com/supabase/agent-skills/tree/main/supabase) | Routes any Supabase task to the right sub-skill. First-line trigger for "DB / Auth / Storage / Realtime / Edge Functions". |
| `supabase:supabase-postgres-best-practices` | `supabase/agent-skills` | [skills.sh/supabase/agent-skills/supabase-postgres-best-practices](https://skills.sh/supabase/agent-skills/supabase-postgres-best-practices) | unreachable — fallback: [github.com/supabase/agent-skills/tree/main/supabase-postgres-best-practices](https://github.com/supabase/agent-skills/tree/main/supabase-postgres-best-practices) | Postgres performance, schema patterns, **RLS** — the auth boundary in a Supabase app. Critical for writing safe migrations. |
| `supabase-developer` | local Supabase plugin (not on public marketplace as standalone) | not on skills.sh (covered by `supabase:supabase` meta-router) | unreachable — fallback: install the Supabase plugin instead via [github.com/supabase/agent-skills](https://github.com/supabase/agent-skills) | Full-stack Supabase: Auth, Storage, Realtime, Edge Functions, RLS — the wider how-to. |
| `vercel:vercel-functions` | `vercel/vercel-plugin` | [skills.sh/vercel/vercel-plugin/vercel-functions](https://skills.sh/vercel/vercel-plugin/vercel-functions) | unreachable — fallback: [github.com/vercel/vercel-plugin](https://github.com/vercel/vercel-plugin) | Route handlers, server actions, edge vs node runtimes, streaming. The execution environment for almost every endpoint. |
| `vercel:vercel-storage` | `vercel/vercel-plugin` | [skills.sh/vercel/vercel-plugin/vercel-storage](https://skills.sh/vercel/vercel-plugin/vercel-storage) | unreachable — fallback: [github.com/vercel/vercel-plugin](https://github.com/vercel/vercel-plugin) | Vercel KV / Blob / Postgres patterns when used alongside Supabase (caches, signed uploads). |
| `vercel:nextjs` | `vercel/vercel-plugin` | [skills.sh/vercel/vercel-plugin/nextjs](https://skills.sh/vercel/vercel-plugin/nextjs) | unreachable — fallback: [github.com/vercel/vercel-plugin](https://github.com/vercel/vercel-plugin) | App Router conventions, server components, route groups, parallel routes. Backend Builder needs this to put handlers in the right place. |
| `vercel:routing-middleware` | `vercel/vercel-plugin` | [skills.sh/vercel/vercel-plugin/routing-middleware](https://skills.sh/vercel/vercel-plugin/routing-middleware) | unreachable — fallback: [github.com/vercel/vercel-plugin](https://github.com/vercel/vercel-plugin) | Auth / rate-limit middleware on the edge. |
| `vercel:env-vars` | `vercel/vercel-plugin` | [skills.sh/vercel/vercel-plugin/env-vars](https://skills.sh/vercel/vercel-plugin/env-vars) | unreachable — fallback: [github.com/vercel/vercel-plugin](https://github.com/vercel/vercel-plugin) | Read-only reference — Backend reads env-var names, doesn't rotate them (DevOps owns rotation). |
| `vercel:ai-sdk` | `vercel/vercel-plugin` | [skills.sh/vercel/vercel-plugin/ai-sdk](https://skills.sh/vercel/vercel-plugin/ai-sdk) | unreachable — fallback: [github.com/vercel/vercel-plugin](https://github.com/vercel/vercel-plugin) | If the product calls models server-side via the Vercel AI SDK rather than the raw Anthropic SDK. |
| `playwright-e2e-testing` | `currents-dev/playwright-best-practices-skill` | [skills.sh/currents-dev/playwright-best-practices-skill/playwright-best-practices](https://skills.sh/currents-dev/playwright-best-practices-skill/playwright-best-practices) | unreachable — fallback: [github.com/currents-dev/playwright-best-practices-skill](https://github.com/currents-dev/playwright-best-practices-skill) | Route-level integration tests; webhook fixture replay. |
| `accessibility-a11y` | `mindrally/skills` | [skills.sh/mindrally/skills/accessibility-a11y](https://skills.sh/mindrally/skills/accessibility-a11y) | unreachable — fallback: [github.com/mindrally/skills](https://github.com/mindrally/skills) | Awareness only — backend rarely owns a11y, but server-rendered HTML and email markup do hit a11y rules. |
| `simplify` | bundled with Claude Code | n/a (bundled — no marketplace import) | n/a (bundled) | Run before every PR — review your own diff for reuse, dead code, over-abstracted layers. |
| `review` | bundled with Claude Code | n/a (bundled) | n/a (bundled) | Self-review pass before marking PR ready. |
| `security-review` | bundled with Claude Code | n/a (bundled) | n/a (bundled) | Run on any PR that touches auth, payments, RLS, or webhook handlers. |
| `init` | bundled with Claude Code | n/a (bundled) | n/a (bundled) | When seeding a fresh repo's `CLAUDE.md` — usually only the first time. |

### Needs installation

| Skill | Source | skills.sh | clawhub.io | Why |
|---|---|---|---|---|
| `stripe/stripe-best-practices` | `stripe/ai` | [skills.sh/stripe/ai/stripe-best-practices](https://skills.sh/stripe/ai/stripe-best-practices) | unreachable (cert/domain expired May 2026) — fallback: [github.com/stripe/ai/tree/main/skills/stripe-best-practices](https://github.com/stripe/ai/tree/main/skills/stripe-best-practices) (community-republished at [clawhub.ai/ifoster01/stripe-best-practices](https://clawhub.ai/ifoster01/stripe-best-practices) — different author, vet before use) | Stripe is the assumed payments layer. Skill covers Checkout vs PaymentIntents, restricted keys, webhook security, idempotency, latest API version (`2026-04-22.dahlia`). Trigger phrases: "stripe", "checkout", "subscription", "webhook", "billing". |
| `stripe/upgrade-stripe` | `stripe/ai` | search [skills.sh/stripe/ai](https://skills.sh/stripe/ai) | unreachable — fallback: [github.com/stripe/ai](https://github.com/stripe/ai) | SDK + API-version upgrades when the workspace pins an older Stripe version. Install once a Stripe upgrade is on the backlog, not before. |
| `better-auth/best-practices` | `better-auth/skills` | [skills.sh/better-auth/skills/better-auth-best-practices](https://skills.sh/better-auth/skills/better-auth-best-practices) | unreachable — fallback: [github.com/better-auth/skills](https://github.com/better-auth/skills) | Install **only if** the workspace uses Better Auth. Covers `BETTER_AUTH_SECRET`, DB adapters (drizzle/prisma/mongodb), session config, plugin selection. If the workspace uses Clerk instead, skip this and rely on `vercel:auth`. |
| `better-auth/email-and-password`, `better-auth/organization`, `better-auth/two-factor` | `better-auth/skills` | search under [skills.sh/better-auth/skills](https://skills.sh/better-auth/skills) | unreachable — fallback: [github.com/better-auth/skills](https://github.com/better-auth/skills) | Add on demand when the matching feature lands on the backlog. |
| `webapp-testing` (Anthropic) | `anthropics/skills/webapp-testing` | [skills.sh/anthropics/skills/webapp-testing](https://skills.sh/anthropics/skills/webapp-testing) | unreachable — fallback: [github.com/anthropics/skills/tree/main/webapp-testing](https://github.com/anthropics/skills/tree/main/webapp-testing) | Anthropic-flavored Playwright skill, complements `playwright-e2e-testing`. |
| `mcp-builder` (Anthropic) | `anthropics/skills/mcp-builder` | [skills.sh/anthropics/skills/mcp-builder](https://skills.sh/anthropics/skills/mcp-builder) | unreachable — fallback: [github.com/anthropics/skills/tree/main/mcp-builder](https://github.com/anthropics/skills/tree/main/mcp-builder) | Install when Backend Builder needs to wrap a new internal API as an MCP for other agents. Not day one. |
| House skill: `workspace-migration-rules` | author via `skill-creator` | n/a — author locally, no marketplace URL | n/a — author locally | Encodes *this workspace's* migration tool, naming convention (`<utc>_<verb>_<thing>.sql`), forbidden patterns (`DROP TABLE` without an explicit issue), and rollback expectations. **Mark "to author via skill-creator".** |

> Do not pre-attach the Anthropic creative suite, the Vercel design skills, or any frontend skill — Backend Builder must not be tempted into UI work.

## MCP servers (`agent.mcp_config`)

| MCP | Scope | Why |
|---|---|---|
| **github** (`ghcr.io/github/github-mcp-server`) | open PRs · comment on PRs · read code/issues. **No merge, no admin, no force-push.** | Primary deliverable surface. The npm `@modelcontextprotocol/server-github` package is unsupported as of April 2025 — use the official Docker image. ([source](https://github.com/github/github-mcp-server)) |
| **supabase** (`@supabase/mcp-server-supabase@latest`) | **`--read-only` always** + `--project-ref=<ref>` scoped to the workspace project. Schema writes happen only via migration files in the PR; never via `apply_migration` from the agent. | Read-only Postgres user prevents accidental data mutation; auditability + rollback live in git, not in MCP call history. ([source: read-only flag](https://github.com/supabase-community/supabase-mcp), [discussion](https://github.com/orgs/supabase/discussions/34325)) |
| **sentry** (`@sentry/mcp-server`) | read-only — issues, events, stack traces. No release-create, no project-create. | Debugging real production errors when the issue says "users report 500 on /api/checkout". ([source](https://github.com/getsentry/sentry-mcp)) |
| **stripe** (`@stripe/mcp` via `npx`) | **test-mode key only**. Read products/prices/customers, simulate webhooks. **Never** attach a live key to a builder agent. | Quickly verify integration shapes without a browser round-trip. ([source](https://www.npmjs.com/package/@stripe/mcp)) |
| **context7** | read-only library docs | Authoritative version-current docs for Next.js, Drizzle, Better Auth, Stripe SDK — beats web search for library questions. |
| **filesystem** | scoped to the task `work_dir` only (`~/multica_workspaces/<task-id>/`) | Standard. Never broaden to `$HOME` or repo root outside the task. |

### How to wire this in Multica (no MCP UI page yet)

The cleaned, spec-valid config lives at `agents/backend-builder/mcp.json` (sibling file). Multica's agent-settings GUI does not expose an MCP page today — [PR #1221](https://github.com/multica-ai/multica/pull/1221) is still open as of May 2026. The daemon DOES however read `agent.mcp_config` from the DB and pass it to Claude Code via `--mcp-config <tempfile>` (PR #1168, shipped v0.2.6). So:

1. **Load `mcp.json` into the agent's `mcp_config` column** — via REST API, `multica` CLI, or direct SQL. Recipes in `agents/MCP-WIRING.md` Part A2. Custom arguments cannot inject `--mcp-config` (the daemon blocks it via `claudeBlockedArgs`).

2. **In the Multica GUI** (Settings → Agents → Backend Builder):
   - **Custom arguments**: leave empty, or use for non-MCP tuning (e.g. `--permission-mode acceptEdits`).
   - **Environment** (one `KEY=VALUE` per line — secrets used as `${VAR}` in `mcp.json`):
     ```
     GITHUB_PAT_BACKEND_BUILDER=ghp_...
     SUPABASE_ACCESS_TOKEN_RO=sbp_...
     SUPABASE_PROJECT_REF=xxxx
     SENTRY_READ_TOKEN=snry_...
     ANTHROPIC_API_KEY=sk-ant-...
     STRIPE_TEST_SECRET_KEY=sk_test_...
     MULTICA_TASK_WORKDIR=/home/<user>/multica_workspaces/<task-id>
     ```
     `MULTICA_TASK_WORKDIR` is the per-task working directory — set it once on the daemon host (it's the same for every task because Multica re-creates the dir per task; in practice you can use the parent `~/multica_workspaces` and the filesystem MCP will see the active task's tree under cwd). If filesystem-MCP scoping is critical, use the broader workspace root.

Rationale notes that previously lived in narrative around the JSON:
- **github** PAT toolsets exclude `actions`, `secrets`, `admin`. No `merge_pull_request`. Defense in depth: even if the agent tries to call those tools, the token can't authorize them.
- **supabase** `--read-only` only governs DB tools (`execute_sql`, `apply_migration`); project/branch-mutating tools remain callable per [Supabase #112](https://github.com/supabase-community/supabase-mcp/issues/112), so the access token is also narrow-scoped.
- **stripe** uses test-mode key only. Never attach a live key here.
- **filesystem** is scoped to `${MULTICA_TASK_WORKDIR}`. Never broaden to `$HOME`.

## Coherence check

The skill set is server-only (Supabase, Vercel Functions, Stripe, Better Auth, testing, simplify) — no frontend skill is attached, which keeps the role honest. The MCP set is biased read: GitHub can only open PRs (not merge), Supabase is read-only with schema changes flowing through migration files in those PRs, Sentry and Stripe are read-only / test-mode. The single write path to production data is `human reviews PR → human merges → CI runs migration` — exactly the auditability + rollback story migrations exist for.
