# Workspace Context — Template

> This is the file you copy **per Multica workspace** and paste into the workspace's `workspace.context` field (Multica → Settings → Workspace → Context). Multica prepends this text to every agent's system prompt on every task — so it is the place where workspace-specific facts live, and the place every agent in the roster (`agents/<name>/`) reads from.
>
> The agent profiles in this folder are intentionally **workspace-agnostic**: same 6 Tier-1 agents drop into a company project, a personal project, or a future commercial product. The thing that changes per workspace is **this file**.
>
> **How to use**:
> 1. Copy this file → `WORKSPACE-CONTEXT.<workspace-slug>.md` (do not edit the template itself; keep it as a clean template).
> 2. Fill in `[FILL IN]` placeholders.
> 3. Delete sections marked `(optional — skip if …)` that don't apply.
> 4. Paste the rendered content into Multica's `workspace.context` field.
> 5. Re-paste whenever the file changes (Multica reads this once per task; updates only land on new tasks, not running ones — `multica.md` §7).

---

## 1. Identity

- **Workspace name**: `[FILL IN]` (e.g. *Acme*, *personal-tinkering*, *research-q3-2026*)
- **Workspace type**: `[coding-product | research-project | personal-misc | internal-tool | other]`
- **Purpose** (1–3 lines): `[FILL IN — what this workspace is for, who reads its outputs, what "done" looks like]`
- **Owner**: `[FILL IN — your name + handle, e.g. troman / t.roman23@outlook.com]`
- **Published externally?** `[yes | no]` *(if **no**, skip §5 entirely)*

---

## 2. Stack

> Defaults below reflect the skills installed on the daemon machine (Next.js + Vercel + Supabase + Tailwind v4 + shadcn). Override per workspace.

- **Language**: TypeScript
- **Framework**: Next.js 16 (App Router, RSC where it pays off — see `vercel:nextjs`)
- **Hosting / deploy**: Vercel (preview per PR; production behind `main`)
- **Database**: Supabase Postgres 17 + RLS (row-level security on every table touching user data)
- **Auth**: `[FILL IN — Clerk | Better Auth | Supabase Auth | NextAuth | none-yet]`
- **Payments**: `[Stripe | none-yet | not-applicable]` *(test-mode key only on builder agents; live keys never reach an agent)*
- **Email transactional**: `[Resend | Postmark | Loops | none-yet]`
- **Analytics / product**: `[Posthog | Vercel Analytics | none-yet]`
- **Error monitoring**: `[Sentry | none-yet]`
- **CSS / UI**: Tailwind v4 (`@theme inline` pattern) + shadcn/ui
- **Tests**: Playwright e2e + the framework-native unit runner
- **Package manager**: `[pnpm | npm | yarn | bun]`
- **Repo URL**: `[FILL IN — github.com/<org>/<repo>]`
- **CLAUDE.md path**: `<repo-root>/CLAUDE.md` *(the Backend/Frontend/Reviewer agents defer to this for stack-specific rules; if missing, run `/init` once to generate one)*

---

## 3. Conventions

- **Branch naming**: `<agent-name>/<issue-id>-<short-slug>` (e.g. `backend-builder/MUL-42-stripe-webhook`)
- **Commit message format**: Conventional Commits (`feat:`, `fix:`, `chore:`, `docs:`, `refactor:`, `test:`)
- **PR rules**: scoped to one issue, ≤400 LOC unless explicitly justified, description includes *what changed / why / how to test / rollback plan*
- **Code style**: defer to `CLAUDE.md` and the workspace's lint/format config (Prettier + ESLint flat config). Reviewers do not flag formatting; they flag missing config.
- **Test runner**: `[FILL IN — vitest | jest | bun test | other]`
- **Migration tool**: `[FILL IN — Supabase migrations | Drizzle Kit | Prisma migrate | raw SQL]` — schema changes always land as a migration file in the PR, never via direct DB write
- **Secrets**: `.env.local` is git-ignored; production secrets live in Vercel / Supabase env settings; agents never log secret values

---

## 4. Non-negotiables (security / privacy / safety)

- **Production database**: read-only from any agent. Writes only via reviewed migration PRs that a human merges.
- **Live billing / payment data**: agents use Stripe **test-mode keys only**. Live keys never appear in any agent's `agent.environment`.
- **PII**: never log email addresses, names, payment details, or auth tokens to stdout. Sentry/Posthog scrubbing rules in `CLAUDE.md`.
- **External shares**: agents never upload code/data to third-party diagram renderers, pastebins, or AI playgrounds (system prompt enforces this).
- **Force-push, --no-verify, history rewrites**: never. If a hook fails, fix the root cause.
- `[ADD WORKSPACE-SPECIFIC RULES HERE — e.g. "no LLM-generated SQL in migrations", "no Stripe Connect changes without human review"]`

---

## 5. Public-facing identity *(optional — skip if "Published externally?" = no)*

Only fill this section if a Content Writer / Designer / Social Media agent is active in this workspace. For purely internal or personal projects, delete the whole section.

- **Product name**: `[FILL IN]`
- **Public URL**: `[FILL IN]`
- **Audience / ICP** (1 sentence): `[FILL IN]`
- **Voice attributes** (pick 3): `[e.g. terse, opinionated, code-heavy, warm, plainspoken, irreverent]`
- **Banned phrases** (5): `[e.g. "leverage", "revolutionize", "in today's fast-paced world", "seamless", "harness the power of"]`
- **Reference brands for tone** (2): `[e.g. Vercel + Linear, or Stripe Press + Notion blog, or Substack onboarding]`
- **Domains to avoid as sources** when researching/writing: `[e.g. listicle SEO sites, AI-summary content farms, Forbes/Inc contributor pieces]`

---

## 6. Active agents in this workspace

> Multica is multi-tenant. Each workspace activates a subset of the roster — disable agents that don't apply.

| Agent | Default for this workspace? | Workspace-specific notes / extra anti-scope |
|---|---|---|
| Chief of Staff | yes (always) | `[FILL IN — e.g. "for personal-misc workspace, skip the weekly digest Autopilot — daily standup only"]` |
| Backend Builder | yes if `repo URL` set | — |
| Frontend Builder | yes if frontend exists | `[skip if backend-only repo]` |
| Reviewer | yes (always) | runs on Codex CLI for model diversity (see `agents/reviewer/instructions.md`) |
| Researcher | yes (always) | `[FILL IN — e.g. "for company workspace, focus on competitive + technical; for personal, technical only"]` |
| Content Writer | yes only if §5 filled in | otherwise disable — there's no audience to write for |

When an agent is disabled, do not delete its config in Multica — set it inactive at the workspace level. Same agent profile can be re-activated for another workspace without re-configuring.

---

## 7. External references

- **Design tokens**: `[path-or-figma-url]`
- **API documentation**: `[swagger-url | openapi-file-path | "see CLAUDE.md"]`
- **Knowledge base**: `[Notion URL | none]`
- **Linked tracker** (if not Multica itself): `[Linear project | GitHub Issues | none]`
- **Status / runbook**: `[Notion URL | path | none]`

---

## 8. Sacred environments (an agent must never touch)

- **Production database**: read-only access via the Supabase REST/Postgres APIs using a **read-only role** scoped to the project; writes blocked at the token level. Schema changes only via migration files in PRs.
- **Live Stripe**: agents have test-mode keys only. The live key lives in Vercel env, never in `agent.environment`.
- **Customer support inbox**: `[FILL IN if applicable — e.g. Plain.com — agent has read access only, replies queued for human approval]`
- **DNS / domain registrar**: never accessed by any agent.
- `[ADD WORKSPACE-SPECIFIC HERE]`

---

## 9. Workspace-specific facts the agents should know

> Free-form. Anything you'd tell a new human contractor on day one belongs here. Examples:
> - "We use a monorepo with `apps/web` (Next.js) and `apps/api` (Hono on Cloudflare Workers)."
> - "All RLS policies must include an `auth.uid() = user_id` check; this is non-negotiable for compliance."
> - "Migrations run via GitHub Actions on merge to `main`; agents do not run them locally."
> - "We don't use feature flags yet; ship behind `main` only."
> - "The blog is in-repo MDX under `apps/web/content/posts/`, not a separate CMS."

`[FILL IN]`

---

## 10. Sources / cross-references

- `agents/<name>/instructions.md` — per-agent role + system prompt; assumes this file is loaded.
- `agents/<name>/skills.md` — per-agent skill list + Tools & API access (CLIs, REST endpoints, env vars, scope).
- `agents/READY-TO-CONFIGURE.md` — index of all 6 Tier-1 agents and the configuration sequence.
- `research/multica.md` §5, §7 — workspace.context schema + how it's prepended to every agent's prompt.
- `research/agent-roster.md` §4–§5 — full roster spec.
