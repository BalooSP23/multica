# Skills + tools for Backend Builder

## Skills (Multica → `agent_skill` table)

Every skill below is imported into Multica via `multica skill import --url <skills.sh URL>`. Every agent task spawns in a fresh `work_dir`, so the daemon must inject every attached skill regardless of what's on the operator's laptop.

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

### To author via `skill-creator`

| Skill (proposed) | Purpose |
|---|---|
| `workspace-migration-rules` | Encodes *this workspace's* migration tool, naming convention (`<utc>_<verb>_<thing>.sql`), forbidden patterns (`DROP TABLE` without an explicit issue), and rollback expectations. |

**Skills bundled with Claude Code** (auto-load when runtime is `claude-code`): `simplify`, `review`, `security-review`, `init`. Use them on every PR self-review pass; `security-review` is mandatory on any auth/payments/RLS/webhook touch.

> Do not pre-attach the Anthropic creative suite, the Vercel design skills, or any frontend skill — Backend Builder must not be tempted into UI work.

## Tools & API access

**Hard rule for this agent**: writes flow through `git` + `gh pr create`, never through direct API mutations. Schema changes flow through migration files in the PR, not through ad-hoc SQL.

| Service | How the agent uses it | Required env vars | Scope |
|---|---|---|---|
| **GitHub** | `gh` CLI for branches, commits, `gh pr create`, `gh issue comment`. The agent never `gh pr merge`s. | `GH_TOKEN=$GITHUB_PAT_BACKEND_BUILDER` (`contents:write`, `pull_requests:write` without merge, `issues:read`) | **No merge, no admin, no force-push.** Branch protection on `main` is the last line of defense. |
| **Supabase** | `supabase` CLI for `db diff`, `db lint`, `db push --dry-run`, generating migration files. Read queries via `psql` against the read-only role. | `SUPABASE_ACCESS_TOKEN=$SUPABASE_ACCESS_TOKEN_RO` (read-only project token) · `SUPABASE_PROJECT_REF=xxxx` · `SUPABASE_DB_URL_RO=postgresql://readonly:...` | **Read-only direct DB access.** Schema mutations only via committed migration files; CI applies them. |
| **Sentry** | Sentry REST API: `curl https://sentry.io/api/0/organizations/<org>/issues/?query=is:unresolved`. | `SENTRY_AUTH_TOKEN=$SENTRY_READ_TOKEN` (org-level read scope) | Read-only — issues, events, stack traces. |
| **Stripe** | `stripe` CLI for `stripe listen` (webhook fixtures), `stripe products list`, `stripe trigger`. SDK calls in code go through the test-mode key only at dev time. | `STRIPE_SECRET_KEY=$STRIPE_TEST_SECRET_KEY` (test-mode `sk_test_…` only — **never live**) | Test-mode only. |
| **Anthropic / Vercel AI SDK** | If the product calls models server-side, the agent edits code that uses these SDKs; runtime API key resolution stays in app config. | `ANTHROPIC_API_KEY` (only if needed for local agent runs of the product code) | Whatever the product needs; the agent doesn't hold prod keys. |
| **Filesystem** | The agent reads/writes inside the per-task `work_dir` (`~/multica_workspaces/<task-id>/`). | `MULTICA_TASK_WORKDIR` (set by the daemon) | Task-scoped only. |
| **Library docs** | When the model needs current Next.js / Drizzle / Better Auth / Stripe SDK docs, it uses Claude Code's WebFetch tool against the official docs site. | none | n/a |

### Wiring in Multica

In the Multica GUI (Settings → Agents → Backend Builder), set:

- **Custom arguments**: leave empty, or use for non-MCP tuning (e.g. `--permission-mode acceptEdits`).
- **Environment** (one `KEY=VALUE` per line):
  ```
  GH_TOKEN=ghp_...
  SUPABASE_ACCESS_TOKEN=sbp_...
  SUPABASE_PROJECT_REF=xxxx
  SUPABASE_DB_URL_RO=postgresql://readonly:...
  SENTRY_AUTH_TOKEN=snry_...
  STRIPE_SECRET_KEY=sk_test_...
  ANTHROPIC_API_KEY=sk-ant-...
  MULTICA_TASK_WORKDIR=/home/<user>/multica_workspaces/<task-id>
  ```

The daemon spawns the agent CLI with these env vars set; the agent invokes `gh`, `supabase`, `stripe`, `psql`, `curl` from Bash. No MCP server config to maintain.

Rationale notes:
- **GitHub** PAT excludes `actions`, `secrets`, `admin`. No `merge_pull_request`. Defense in depth — even if the agent tries, the token can't authorize it.
- **Supabase** read-only role + read-only project token means accidental data mutations are impossible at the credential layer; auditability + rollback live in git.
- **Stripe** test-mode key only. A live key in this env field is a P0 incident.
- **Filesystem** is scoped to `${MULTICA_TASK_WORKDIR}`. Never broaden to `$HOME`.

## Coherence check

The skill set is server-only (Supabase, Vercel Functions, Stripe, Better Auth, testing) — no frontend skill is attached, which keeps the role honest. The API set is biased read: GitHub can only open PRs (not merge), Supabase is read-only with schema changes flowing through migration files, Sentry and Stripe are read-only / test-mode. The single write path to production data is `human reviews PR → human merges → CI runs migration` — exactly the auditability + rollback story migrations exist for.
