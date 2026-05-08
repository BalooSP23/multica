# Skills + MCPs for Backend Builder

## Skills (Multica → `agent_skill` table)

Every skill below is imported into Multica via `multica skill import --url <skills.sh URL>`. The "Already installed locally" / "Needs installation" distinction doesn't apply to Multica — every agent task spawns in a fresh `work_dir`, so the daemon must inject every attached skill regardless of what's on the operator's laptop.

### To attach (import from skills.sh)

| Skill | Why for this role | skills.sh | GitHub fallback |
|---|---|---|---|
| `claude-api` | Backend touches the Anthropic SDK any time the product calls a model server-side; canonical reference for prompt caching, tool use, model migrations. | [skills.sh/anthropics/skills/claude-api](https://skills.sh/anthropics/skills/claude-api) | [github.com/anthropics/skills/tree/main/claude-api](https://github.com/anthropics/skills/tree/main/claude-api) |
| `supabase` | Meta-router for Supabase tasks. First-line trigger for "DB / Auth / Storage / Realtime / Edge Functions". | [skills.sh/supabase/agent-skills/supabase](https://skills.sh/supabase/agent-skills/supabase) | [github.com/supabase/agent-skills/tree/main/supabase](https://github.com/supabase/agent-skills/tree/main/supabase) |
| `supabase-postgres-best-practices` | Postgres performance, schema patterns, **RLS** — the auth boundary in a Supabase app. Critical for safe migrations. | [skills.sh/supabase/agent-skills/supabase-postgres-best-practices](https://skills.sh/supabase/agent-skills/supabase-postgres-best-practices) | [github.com/supabase/agent-skills/tree/main/supabase-postgres-best-practices](https://github.com/supabase/agent-skills/tree/main/supabase-postgres-best-practices) |
| `vercel-functions` | Route handlers, server actions, edge vs node runtimes, streaming. Execution environment for almost every endpoint. | [skills.sh/vercel/vercel-plugin/vercel-functions](https://skills.sh/vercel/vercel-plugin/vercel-functions) | [github.com/vercel/vercel-plugin](https://github.com/vercel/vercel-plugin) |
| `vercel-storage` | Vercel KV / Blob / Postgres patterns alongside Supabase (caches, signed uploads). | [skills.sh/vercel/vercel-plugin/vercel-storage](https://skills.sh/vercel/vercel-plugin/vercel-storage) | [github.com/vercel/vercel-plugin](https://github.com/vercel/vercel-plugin) |
| `nextjs` (Vercel) | App Router conventions, server components, route groups, parallel routes. | [skills.sh/vercel/vercel-plugin/nextjs](https://skills.sh/vercel/vercel-plugin/nextjs) | [github.com/vercel/vercel-plugin](https://github.com/vercel/vercel-plugin) |
| `routing-middleware` | Auth / rate-limit middleware on the edge. | [skills.sh/vercel/vercel-plugin/routing-middleware](https://skills.sh/vercel/vercel-plugin/routing-middleware) | [github.com/vercel/vercel-plugin](https://github.com/vercel/vercel-plugin) |
| `env-vars` (Vercel) | Read-only reference — Backend reads env-var names, doesn't rotate them (DevOps owns rotation). | [skills.sh/vercel/vercel-plugin/env-vars](https://skills.sh/vercel/vercel-plugin/env-vars) | [github.com/vercel/vercel-plugin](https://github.com/vercel/vercel-plugin) |
| `ai-sdk` (Vercel) | If the product calls models server-side via the Vercel AI SDK rather than the raw Anthropic SDK. | [skills.sh/vercel/vercel-plugin/ai-sdk](https://skills.sh/vercel/vercel-plugin/ai-sdk) | [github.com/vercel/vercel-plugin](https://github.com/vercel/vercel-plugin) |
| `playwright-best-practices` | Route-level integration tests; webhook fixture replay. | [skills.sh/currents-dev/playwright-best-practices-skill/playwright-best-practices](https://skills.sh/currents-dev/playwright-best-practices-skill/playwright-best-practices) | [github.com/currents-dev/playwright-best-practices-skill](https://github.com/currents-dev/playwright-best-practices-skill) |
| `accessibility-a11y` | Awareness only — backend rarely owns a11y, but server-rendered HTML and email markup do hit a11y rules. | [skills.sh/mindrally/skills/accessibility-a11y](https://skills.sh/mindrally/skills/accessibility-a11y) | [github.com/mindrally/skills](https://github.com/mindrally/skills) |
| `webapp-testing` | Anthropic-flavored Playwright skill, complements `playwright-best-practices`. | [skills.sh/anthropics/skills/webapp-testing](https://skills.sh/anthropics/skills/webapp-testing) | [github.com/anthropics/skills/tree/main/webapp-testing](https://github.com/anthropics/skills/tree/main/webapp-testing) |
| `stripe-best-practices` | Checkout vs PaymentIntents, restricted keys, webhook security, idempotency. Triggers: "stripe", "checkout", "subscription", "webhook", "billing". | [skills.sh/stripe/ai/stripe-best-practices](https://skills.sh/stripe/ai/stripe-best-practices) | [github.com/stripe/ai](https://github.com/stripe/ai) |
| `better-auth-best-practices` | Install **only if** the workspace uses Better Auth. Covers `BETTER_AUTH_SECRET`, DB adapters, session config, plugin selection. | [skills.sh/better-auth/skills/better-auth-best-practices](https://skills.sh/better-auth/skills/better-auth-best-practices) | [github.com/better-auth/skills](https://github.com/better-auth/skills) |
| `mcp-builder` | Install when Backend Builder needs to wrap a new internal API as an MCP for other agents. Not day one. | [skills.sh/anthropics/skills/mcp-builder](https://skills.sh/anthropics/skills/mcp-builder) | [github.com/anthropics/skills/tree/main/mcp-builder](https://github.com/anthropics/skills/tree/main/mcp-builder) |

### To author via `skill-creator`

| Skill (proposed) | Purpose |
|---|---|
| `workspace-migration-rules` | Encodes *this workspace's* migration tool, naming convention (`<utc>_<verb>_<thing>.sql`), forbidden patterns (`DROP TABLE` without an explicit issue), and rollback expectations. |

**Skills bundled with Claude Code** (no marketplace import; auto-load when runtime is `claude-code`): `simplify`, `review`, `security-review`, `init`. Use them on every PR self-review pass; `security-review` is mandatory on any auth/payments/RLS/webhook touch.

> Do not pre-attach the Anthropic creative suite, the Vercel design skills, or any frontend skill — Backend Builder must not be tempted into UI work.

## MCP servers (`agent.mcp_config`)

| MCP | Scope | Why |
|---|---|---|
| **github** (`ghcr.io/github/github-mcp-server`) | open PRs · comment on PRs · read code/issues. **No merge, no admin, no force-push.** | Primary deliverable surface. The npm `@modelcontextprotocol/server-github` package is unsupported as of April 2025 — use the official Docker image. ([source](https://github.com/github/github-mcp-server)) |
| **supabase** (`@supabase/mcp-server-supabase@latest`) | **`--read-only` always** + `--project-ref=<ref>` scoped to the workspace project. Schema writes happen only via migration files in the PR. | Read-only Postgres user prevents accidental data mutation; auditability + rollback live in git, not MCP call history. ([source](https://github.com/supabase-community/supabase-mcp)) |
| **sentry** (`@sentry/mcp-server`) | read-only — issues, events, stack traces. No release-create, no project-create. | Debugging real production errors when the issue says "users report 500 on /api/checkout". |
| **stripe** (`@stripe/mcp` via `npx`) | **test-mode key only**. Read products/prices/customers, simulate webhooks. **Never** attach a live key. | Verify integration shapes without a browser round-trip. |
| **context7** | read-only library docs | Authoritative version-current docs for Next.js, Drizzle, Better Auth, Stripe SDK — beats web search. |
| **filesystem** | scoped to the task `work_dir` only (`~/multica_workspaces/<task-id>/`) | Standard. Never broaden to `$HOME` or repo root outside the task. |

### How to wire this in Multica

The cleaned, spec-valid config lives at `agents/backend-builder/mcp.json`. Multica's agent-settings GUI does not expose an MCP page today — [PR #1221](https://github.com/multica-ai/multica/pull/1221) is open. The daemon reads `agent.mcp_config` from the DB and passes it to Claude Code via `--mcp-config <tempfile>` (PR #1168, shipped v0.2.6). Full procedure in `agents/MCP-WIRING.md` Part A.

In the Multica GUI (Settings → Agents → Backend Builder):
- **Custom arguments**: leave empty, or use for non-MCP tuning (e.g. `--permission-mode acceptEdits`).
- **Environment** (one `KEY=VALUE` per line — referenced as `${VAR}` in `mcp.json`):
  ```
  GITHUB_PAT_BACKEND_BUILDER=ghp_...
  SUPABASE_ACCESS_TOKEN_RO=sbp_...
  SUPABASE_PROJECT_REF=xxxx
  SENTRY_READ_TOKEN=snry_...
  ANTHROPIC_API_KEY=sk-ant-...
  STRIPE_TEST_SECRET_KEY=sk_test_...
  MULTICA_TASK_WORKDIR=/home/<user>/multica_workspaces/<task-id>
  ```

Rationale notes:
- **github** PAT toolsets exclude `actions`, `secrets`, `admin`. No `merge_pull_request`. Defense in depth: even if the agent tries to call those tools, the token can't authorize them.
- **supabase** `--read-only` only governs DB tools (`execute_sql`, `apply_migration`); project/branch-mutating tools remain callable per [Supabase #112](https://github.com/supabase-community/supabase-mcp/issues/112), so the access token is also narrow-scoped.
- **stripe** uses test-mode key only.
- **filesystem** is scoped to `${MULTICA_TASK_WORKDIR}`. Never broaden to `$HOME`.

## Coherence check

The skill set is server-only (Supabase, Vercel Functions, Stripe, Better Auth, testing) — no frontend skill is attached, which keeps the role honest. The MCP set is biased read: GitHub can only open PRs (not merge), Supabase is read-only with schema changes flowing through migration files, Sentry and Stripe are read-only / test-mode. The single write path to production data is `human reviews PR → human merges → CI runs migration` — exactly the auditability + rollback story migrations exist for.
