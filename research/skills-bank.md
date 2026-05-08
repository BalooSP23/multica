# Skills Bank — Curated for Vibe Coders

> A high-signal catalog of Agent Skills worth installing today. Sourced from the official Anthropic skills repo (129k★), Vercel Labs agent-skills, and the three largest curated awesome lists (`ComposioHQ/awesome-claude-skills` 58k★, `VoltAgent/awesome-agent-skills` 20k★, `travisvn/awesome-claude-skills` 12k★). Each entry is briefly analyzed — not just listed — so you can decide whether to install before reading the SKILL.md yourself.

---

## 1. How to read this list

Every Agent Skill lives at three loading levels (Anthropic's "progressive disclosure" model):

| Level | Loaded when | Cost | Content |
|---|---|---|---|
| **L1 — Metadata** | Always (session start) | ~100 tokens / skill | `name` + `description` from YAML frontmatter |
| **L2 — Body** | When `description` matches user request | <5k tokens | `SKILL.md` body — instructions, examples, guidance |
| **L3 — Bundled files** | On demand by bash | Effectively unlimited | Scripts, references, datasets, schemas |

A "good" skill is one where L1 is precise (so it triggers exactly when needed, no false positives), L2 fits in <5k tokens, and L3 carries the deterministic operations (scripts) so the agent doesn't reinvent them.

**The "be slightly pushy" rule** (from Anthropic's `skill-creator`): Claude tends to **under-trigger** skills. Descriptions should explicitly enumerate trigger phrases and adjacent contexts. Compare:

> ❌ "How to build a simple fast dashboard."
>
> ✅ "How to build a simple fast dashboard. Use this skill whenever the user mentions dashboards, data visualization, internal metrics, or wants to display any kind of company data, even if they don't explicitly ask for a 'dashboard.'"

If a skill you're considering has a vague description, that's a signal it'll either over-trigger or never trigger.

---

## 2. Selection criteria

A skill earns a slot in this bank only if it ticks most of these:

- **Source signal**: published by an official org (Anthropic, Vercel, Stripe, Cloudflare, Supabase, Sentry, etc.) **OR** sits in a curated awesome list ≥10k★ **OR** has direct adoption metrics (installs / weekly downloads).
- **Active maintenance**: last commit within ~6 months.
- **Tight scope**: does one thing, triggers narrowly, doesn't claim to be a Swiss Army knife.
- **Deterministic L3**: includes scripts where the operation is mechanical (form filling, file conversion, validation).
- **Audit-friendly**: no surprise outbound network calls, no opaque binaries.
- **Plays well with progressive disclosure**: SKILL.md is genuinely <5k tokens; long reference content is in separate L3 files.

The bank is **tiered by who needs it first**, not alphabetically. Tier 1 = essentials for anyone using Claude Code. Tier 5 = niche.

A note on the user's existing setup: from the active session reminder, the user already has ~30 skills installed locally. Entries marked **(✓ already installed)** are confirmed in the user's skill list. The rest are recommendations.

---

## 3. Tier 1 — Essentials (everyone)

These earn their L1 slot in any Claude Code setup. Small footprint, broad triggers, durable value.

### `anthropics/skill-creator` 
**Source**: `anthropics/skills/skill-creator` · **License**: Apache-2.0

Author and iterate on other skills with an evaluation loop. Knows how to interview you for intent, draft a SKILL.md, run test prompts, score the results, and tighten the description for triggering accuracy. Includes an `eval-viewer/generate_review.py` script for visual results review.

- **Triggers**: creating skills, modifying skills, evaluating skill performance.
- **Why install**: it's the meta-tool. Even if you only ever write 3 skills, this ensures their descriptions actually trigger.
- **Specifically valuable for**: anyone running Multica (skills are the primary lever) or building team-wide SOPs.

### `anthropics/mcp-builder`
**Source**: `anthropics/skills/mcp-builder` · **License**: Apache-2.0

Build high-quality MCP (Model Context Protocol) servers in Python (FastMCP) or Node/TypeScript (MCP SDK). Walks through API coverage decisions, naming conventions, error-message design, and tool description clarity. Practical guide to the protocol.

- **Triggers**: "build an MCP server", "expose `<service>` to Claude", "wrap `<API>` for agents".
- **Why install**: every time you want to give an agent a new capability, the answer is increasingly an MCP server. This is how to build one well.

### `anthropics/webapp-testing`
**Source**: `anthropics/skills/webapp-testing` · **License**: Apache-2.0

Test local web apps with Playwright. Covers spinning up the dev server, running the test agent against it, capturing screenshots, and reporting failures.

- **Triggers**: "test this UI", "verify this page works", "run e2e".
- **Why install**: most "vibe-coded" UIs ship without tests; this skill makes the test pass before you do.

### `anthropics/pdf` *(✓ already installed)*
**Source**: `anthropics/skills/pdf`

Extract text and tables from PDFs, fill forms, merge documents, OCR scanned pages. Bundles `pdfplumber`-based scripts (L3) — Claude doesn't reinvent PDF parsing each time.

- **Triggers**: any mention of `.pdf`, document extraction, form filling.

### `anthropics/xlsx` *(✓ already installed)*
**Source**: `anthropics/skills/xlsx`

Create, edit, analyze Excel spreadsheets. Charts, formulas, multi-sheet operations.

- **Triggers**: `.xlsx`, "spreadsheet", "Excel".

### `anthropics/docx`
**Source**: `anthropics/skills/docx`

Create, edit, analyze Word documents.

- **Triggers**: `.docx`, "Word document".
- **Pair with**: `doc-coauthoring` for collaborative editing flows.

### `anthropics/pptx`
**Source**: `anthropics/skills/pptx`

Create, edit, analyze PowerPoint presentations.

- **Triggers**: `.pptx`, "slide deck", "presentation".

### `anthropics/claude-api` *(✓ already installed as `claude-api`)*
**Source**: `anthropics/skills/claude-api`

Up-to-date Anthropic SDK reference for 8 languages. Includes prompt-caching guidance, model-version migration help, tool-use patterns, citations, batch API, files API. Bundled with Claude Code.

- **Triggers**: any code importing `anthropic` / `@anthropic-ai/sdk`; Claude API questions; model-version migrations.

---

## 4. Tier 2 — Front-end & design

If you build UIs, this set is the difference between "looks AI-generated" and "looks designed".

### `anthropics/frontend-design` *(✓ already installed via `frontend-design:frontend-design`)*
**Source**: `anthropics/skills/frontend-design` · ~277k installs as of Mar 2026

Generate distinctive, production-grade frontend interfaces that **avoid generic "AI slop" aesthetics**. Pushes you to commit to a bold aesthetic direction (brutalist / editorial / luxury / pastel etc.) before coding. Explicit guidelines on typography (avoid Inter/Roboto/Arial), color theory (dominant + sharp accents over timid even palettes), motion (CSS-first, Motion library for React), spatial composition (asymmetry, overlap, generous negative space), and backgrounds (gradient meshes, noise, grain overlays).

- **Triggers**: "build a landing page", "create a dashboard", "design this component", "make this look better".
- **Why this beats default Claude UI output**: Claude's default has an aesthetic ceiling. This skill explicitly raises it.

### `anthropics/web-artifacts-builder`
**Source**: `anthropics/skills/web-artifacts-builder`

Build complex Claude.ai HTML artifacts with React + Tailwind. Single-file, runnable, visually polished.

- **Triggers**: "make me a quick prototype", "interactive demo", "playable artifact".

### `anthropics/canvas-design`, `anthropics/algorithmic-art`, `anthropics/theme-factory`, `anthropics/brand-guidelines`

The Anthropic creative suite:
- **canvas-design** — visual art in PNG/PDF.
- **algorithmic-art** — generative art with p5.js + seeded randomness (reproducible).
- **theme-factory** — apply professional themes to artifacts or generate custom themes.
- **brand-guidelines** — apply Anthropic's brand colors/typography (template for building your own brand skill).

### `nextlevelbuilder/ui-ux-pro-max-skill` *(✓ already installed as `ui-ux-pro-max:ui-ux-pro-max`)*
**Source**: `nextlevelbuilder/ui-ux-pro-max-skill` · 74k★

Broad UI/UX design intelligence: 50 styles, 21 palettes, 50 font pairings, 20 charts, 9 stacks (React, Next.js, Vue, Svelte, SwiftUI, React Native, Flutter, Tailwind, shadcn). Triggers on a wide vocabulary of design verbs (plan / build / create / design / implement / review / fix / improve / optimize / refactor) across project types (landing page, dashboard, e-commerce, SaaS, portfolio, blog).

- **Why install alongside `frontend-design`**: this one is broader (catalog + reference); `frontend-design` is more opinionated (aesthetic direction). They compose.

### `shadcn/ui` skill *(✓ already installed as `shadcn` and via `vercel:shadcn`)*
**Source**: `shadcn/ui` repo, `skills/shadcn/`

shadcn-aware skill: CLI usage, component composition, custom registries, theming, Tailwind integration, troubleshooting. Knows the v4 + `@theme inline` patterns.

- **Triggers**: "add a shadcn component", "set up shadcn", "shadcn theming".

### Vercel Labs — `web-design-guidelines`, `react-best-practices`, `composition-patterns`, `react-view-transitions`, `vercel-cli-with-tokens`, `deploy-to-vercel`, `react-native-skills`
**Source**: `vercel-labs/agent-skills`

Engineered by the Vercel team. The standout pair:
- **react-best-practices** — TSX-aware reviewer triggered after editing multiple TSX components: component structure, hooks, accessibility, performance, TypeScript patterns. *(✓ already installed as `vercel:react-best-practices`)*
- **web-design-guidelines** — design rules at the Vercel quality bar.

The full set covers React-Native skills, view-transitions API, reusable composition patterns. Lighter-weight than `ui-ux-pro-max` and tightly scoped.

### `tailwind-v4-shadcn` *(✓ already installed)*
**Source**: community / installed locally

Sets up Tailwind v4 with shadcn/ui using the `@theme inline` pattern + CSS-variable architecture. Specifically prevents 8 documented errors (colors not working, `tw-animate-css` errors, `@theme inline` dark-mode conflicts, `@apply` breaking, v3 migration issues).

### `refactoring-ui` *(✓ already installed)*
**Source**: based on Wathan & Schoger's *Refactoring UI* book

Practical UI design system: visual hierarchy, color, typography, layout, depth. Triggers on "make this look better", styling work, UI component improvement.

---

## 5. Tier 3 — Stack-native (pick what you actually use)

Don't install all of these. Pick the matching ones for your stack.

### Vercel ecosystem *(✓ user has the full Vercel plugin set)*

The user's local Vercel skill set is unusually comprehensive (visible in the session reminder). It already includes `vercel:bootstrap`, `vercel:deploy`, `vercel:env`, `vercel:status`, `vercel:nextjs`, `vercel:next-cache-components`, `vercel:next-upgrade`, `vercel:turbopack`, `vercel:ai-sdk`, `vercel:ai-gateway`, `vercel:vercel-functions`, `vercel:vercel-storage`, `vercel:vercel-cli`, `vercel:deployments-cicd`, `vercel:env-vars`, `vercel:routing-middleware`, `vercel:runtime-cache`, `vercel:vercel-sandbox`, `vercel:vercel-agent`, `vercel:next-forge`, `vercel:chat-sdk`, `vercel:workflow`, `vercel:auth`, `vercel:marketplace`, `vercel:shadcn`, `vercel:react-best-practices`, `vercel:knowledge-update`, `vercel:verification`, `vercel:ai-architect`. **Coverage of the Vercel stack here is essentially complete.**

### Supabase

- **`supabase/postgres-best-practices`** *(✓ already installed as `supabase:supabase-postgres-best-practices`)* — Postgres performance tuning, schema patterns, RLS.
- **`supabase-developer`** *(✓ already installed)* — Full-stack Supabase: Auth, Storage, Realtime, Edge Functions, RLS.
- **`supabase:supabase`** *(✓ already installed)* — meta-router for any Supabase task.

### Auth

- **`better-auth/*`** — 7 skills (best-practices, providers, create-auth, email+password, organization, two-factor, explain-error). Best Auth is the modern Auth.js successor.
- **`vercel:auth`** *(✓ already installed)* — Clerk (native Vercel Marketplace), Descope, Auth0 setup for Next.js.
- **`auth0/*`** — official Auth0 skills.

### Payments

- **`stripe/stripe-best-practices`** — building Stripe integrations correctly.
- **`stripe/upgrade-stripe`** — SDK and API version upgrades.

### Databases & data

- **`mongodb/*`** — official MongoDB.
- **`redis/*`** — official Redis.
- **`tinybirdco/tinybird-best-practices`** — datasources, pipes, endpoints, SQL.
- **ClickHouse skill** — analytics workloads.
- **DuckDB skill** — local analytics.
- **Neon skill** — serverless Postgres.

### Notifications & comms

- **`trycourier/courier-skills`** — multi-channel notifications (email, SMS, push, chat).
- **`anthropics/internal-comms`** — write status reports, newsletters, FAQs.
- **`anthropics/slack-gif-creator`** — animated GIFs sized for Slack.

### Mobile

- **`expo/*`** — official Expo skills.
- **`callstackincubator/react-native-best-practices`** — performance optimization for RN.
- **`callstackincubator/upgrading-react-native`** — RN upgrade workflow with templates and pitfalls.

### Animation & video

- **`gsap` (GreenSock)** — official animation skill.
- **Remotion skill** — programmatic video; ~117k weekly installs, security-audited (Agent Trust Hub + Socket).

### Testing

- **`anthropics/webapp-testing`** *(in Tier 1)*
- **`playwright-e2e-testing`** *(✓ already installed)* — broader Playwright skill: cross-browser, auto-wait, built-in test runner.

### Docs & accessibility

- **`accessibility-a11y`** *(✓ already installed)* — WCAG-compliant interfaces.
- **`anthropics/doc-coauthoring`** — collaborative document editing.

---

## 6. Tier 4 — Specialist tooling

Install when the specific need arises; don't preinstall.

### Security

- **Trail of Bits security skills** — vetted by a top security firm. Audit-style and threat-modeling skills.
- **`security-review`** *(✓ already installed)* — security review of pending changes on a branch.

### Observability & review

- **Sentry skills** — error monitoring + debugging integration.
- **Datadog Labs skills** — observability dashboards, metrics, traces.
- **CodeRabbit skill** — automated PR review.
- **`review`** *(✓ already installed)* — review a PR.

### Headless browser & scraping

- **Browserbase skill** — headless browser automation.
- **Firecrawl skill** — web scraping with structured output.

### AI/ML inference

- **Hugging Face skill** — official.
- **fal.ai skill** — fast diffusion inference.
- **Replicate skill** — model hosting.
- **Venice.ai skill** — privacy-focused inference.

### Connectors

- **Composio skill** — connect agents to 1000+ external apps with managed auth. The "I need to integrate `<thing>` quickly" lever.

### Search & web

- **Brave skill** — Brave search APIs.

### Crypto / fintech

- **Coinbase skill**, **Binance skill** — official.

### Vector / GraphQL

- **Apollo GraphQL skill** — official.

---

## 7. Tier 5 — Workflow / non-engineering

Skills for adjacent disciplines. Useful when an agent is helping with marketing, PM, advertising, or comms.

- **Corey Haines (marketing)** — community-published marketing skill set.
- **Dean Peters / Paweł Huryn (product management)** — PM frameworks.
- **Kim Barrett (advertising)** — ad copywriting and campaign skills.
- **Garry Tan / gstack** — startup-stack skills.
- **Addy Osmani (web quality)** — performance + Core Web Vitals.

---

## 8. The user's current local stack — for reference

From the active session list (already installed and active in this Claude Code session):

**Engineering / dev tools** — `claude-api`, `init`, `review`, `security-review`, `simplify`, `fewer-permission-prompts`, `loop`, `schedule`, `keybindings-help`, `update-config`, `find-skills`.

**UI / front-end** — `frontend-design:frontend-design`, `ui-ux-pro-max:ui-ux-pro-max`, `ui-ux-expert`, `refactoring-ui`, `tailwind-v4-shadcn`, `accessibility-a11y`.

**Vercel suite** (full coverage) — `vercel:ai-architect`, `vercel:ai-gateway`, `vercel:ai-sdk`, `vercel:auth`, `vercel:bootstrap`, `vercel:chat-sdk`, `vercel:deploy`, `vercel:deployments-cicd`, `vercel:env`, `vercel:env-vars`, `vercel:knowledge-update`, `vercel:marketplace`, `vercel:next-cache-components`, `vercel:next-forge`, `vercel:next-upgrade`, `vercel:nextjs`, `vercel:react-best-practices`, `vercel:routing-middleware`, `vercel:runtime-cache`, `vercel:shadcn`, `vercel:status`, `vercel:turbopack`, `vercel:vercel-agent`, `vercel:vercel-cli`, `vercel:vercel-functions`, `vercel:vercel-sandbox`, `vercel:vercel-storage`, `vercel:verification`, `vercel:workflow`.

**Supabase suite** — `supabase:supabase`, `supabase:supabase-postgres-best-practices`, `supabase-developer`.

**Document / data** — `pdf`, `xlsx`, `graphify`.

**Testing** — `playwright-e2e-testing`.

**The clear gaps**, given the user is building toward a Multica-style workflow:

| Gap | Skill to add |
|---|---|
| Skill authoring & iteration | `anthropics/skill-creator` |
| MCP server building | `anthropics/mcp-builder` |
| Web-artifact prototyping | `anthropics/web-artifacts-builder` |
| Webapp e2e (anthropic-flavored) | `anthropics/webapp-testing` |
| Stripe integration | `stripe/stripe-best-practices` |
| Better Auth (if not Clerk) | `better-auth/*` set |
| Async / queue workflows beyond Vercel Workflow | `tinybirdco`, Courier (pick by need) |
| Doc editing | `anthropics/docx`, `anthropics/pptx` |
| Internal comms drafting | `anthropics/internal-comms` |

Everything else in the user's stack is already covered.

---

## 9. Skill authoring quickstart

The fastest way to author a working skill:

```bash
mkdir -p ~/.claude/skills/<skill-name>
$EDITOR ~/.claude/skills/<skill-name>/SKILL.md
```

Minimum viable `SKILL.md`:

```markdown
---
name: <skill-name>
description: What it does AND when Claude should use it. Include trigger phrases explicitly. Be slightly pushy about when it should fire — Claude tends to under-trigger. Mention adjacent contexts. (Up to 1024 chars; this is the only thing Claude sees at L1.)
---

# <Skill Title>

## Instructions
[Step-by-step guidance for Claude.]

## Examples
- Concrete example 1
- Concrete example 2

## Guidelines
- Do this.
- Avoid that.
```

**Required field rules**:
- `name`: ≤64 chars, lowercase + numbers + hyphens. **Must not** contain "anthropic" or "claude". No XML.
- `description`: ≤1024 chars, no XML, must be non-empty.

**Test it** by mentioning the skill's domain — Claude should automatically reach for it. If it doesn't, the description is too narrow or too generic. Use `anthropics/skill-creator` to iterate; it has an eval loop built in.

**Add bundled L3 files** (scripts, references) freely:

```
~/.claude/skills/<skill-name>/
├── SKILL.md
├── REFERENCE.md      # detailed reference, only loaded when SKILL.md links it
└── scripts/
    └── do_thing.py   # executed via bash; the source never enters context
```

**For Multica**, use the same format and import via `multica skill import --url <url>` or `multica skill create`. The daemon writes it to the provider-native location at task launch.

---

## 10. Repos to subscribe to (for new skills as they ship)

- **`anthropics/skills`** — official, 129k★. Canonical reference + spec.
- **`vercel-labs/agent-skills`** — production engineering skills from the Vercel team.
- **`VoltAgent/awesome-agent-skills`** — 20k★, curated, *"hand-picked, not AI-slop generated"*. Organized by publishing org (Anthropic, Google Gemini, Stripe, Cloudflare, Sentry, …).
- **`ComposioHQ/awesome-claude-skills`** — 58k★, broad coverage with 1000+ entries.
- **`travisvn/awesome-claude-skills`** — 12k★, foundational curated list.
- **`shadcn/ui` skills/** — official shadcn skill.
- **skills.sh** (https://skills.sh) — primary marketplace; 581+ vendor-official skills. All URLs in this bank are verified live.
- **clawhub.ai** (https://clawhub.ai) — community marketplace referenced by Multica's docs; small (~50 skills), use as last-resort fallback.
- **skills.sh** — second public skill marketplace referenced by Multica imports.
- **officialskills.sh** — VoltAgent's hosted skill index.

---

## 11. Security checklist before installing

Treat installing a skill **like installing software** — Anthropic's own warning, paraphrased:

1. **Audit the SKILL.md and every bundled file** before installing. Look for:
   - Unexpected outbound network calls (especially in scripts).
   - File reads/writes outside the skill's stated scope.
   - Operations that don't match the description.
2. **External-URL fetches at runtime** are the highest-risk pattern. Even a trustworthy skill can be compromised if its external dependency changes. Prefer skills that bundle their references statically.
3. **Tool misuse risk**: a malicious skill can invoke any tool the agent has — file ops, bash, code execution. Constrain agent permissions accordingly.
4. **Data exposure risk**: skills with access to sensitive data could be designed to leak it. Don't auto-install in a workspace with production credentials.
5. **Prefer official orgs** over solo authors unless the solo author is well-known and the repo is well-starred.
6. **Re-audit on update**. A skill you trusted last month may have been replaced.

For a Multica self-hosted workspace, the threat model is sharper because skills are injected into every task working directory at launch. Treat the skill bank as part of the trusted compute base.

---

## 12. Sources

- Anthropic, *Agent Skills* spec — https://platform.claude.com/docs/en/docs/agents-and-tools/agent-skills/overview
- `anthropics/skills` (129k★) — https://github.com/anthropics/skills
- Anthropic best-practices guide — https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices
- `vercel-labs/agent-skills` — https://github.com/vercel-labs/agent-skills
- `VoltAgent/awesome-agent-skills` (20.5k★) — https://github.com/VoltAgent/awesome-agent-skills
- `ComposioHQ/awesome-claude-skills` (58.4k★) — https://github.com/ComposioHQ/awesome-claude-skills
- `travisvn/awesome-claude-skills` (12.2k★) — https://github.com/travisvn/awesome-claude-skills
- `nextlevelbuilder/ui-ux-pro-max-skill` (74k★) — https://github.com/nextlevelbuilder/ui-ux-pro-max-skill
- `shadcn/ui` — https://github.com/shadcn/ui
- Skill marketplaces — skills.sh (primary) · officialskills.sh (vendor-official mirror) · clawhub.ai (community)
- Cross-references: `multica.md` (how Multica injects skills into provider-native paths), `agent-orchestration.md` (which skills support which orchestration patterns)
