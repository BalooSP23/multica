# Agent Roster — The Solo Founder's AI Team

> Companion to `agent-orchestration.md` (when to add agents, what pattern) and `multica.md` (the coordination layer). This file answers the next question: **which agents do I actually configure, in what order, with what scope, API access, and skills, to cover SaaS engineering, research, marketing, social media, sales, and ops as one human?**
>
> All recommendations are constrained by the rules already established: ≤5 sub-agents in parallel, multi-agent costs ~15× single chat, sub-agents need 4-part briefs, one-agent-per-role-not-per-project, skills + API tokens are the per-agent levers.

---

## TL;DR — five rules that frame the rest

1. **Agents are roles, not concurrent runners.** You can configure 20 agents in Multica and only ever have 3–5 running at once. The roster is a *bench*; the daemon dispatches when issues land. (`MULTICA_DAEMON_MAX_CONCURRENT_TASKS` defaults to 20 — that's the upper bound on parallelism, not headcount.)
2. **Hire in tiers.** Day-one needs ~6 agents. Post-launch ~12. A mature solo studio sits around ~18. Beyond that, you're managing the team instead of the work.
3. **One agent = one role + one set of API credentials + one skill set + one system prompt.** If two of those four diverge enough to need different config, that's two agents. If they don't, it's one with a clearer brief.
4. **The human is head-of-staff, not head-of-department.** You set direction, triage agent comments, approve PRs, and decide tradeoffs. Recurring cadence (review reminders, dep audits, content calendars, weekly KPI reports) belongs in **Autopilots**, not in your calendar.
5. **Engineering work is the easy part to staff.** The roster's leverage shows up in non-engineering roles — research, marketing, social, support — where solo founders chronically underspend attention. The agent team is what lets you run a real go-to-market without hiring.

Skip to §4 for the full roster, §5 for per-agent specs, §6 for scenario playbooks, §8 for the week-by-week hire order.

---

## 1. The framing — agents are roles, not runners

Three traps to avoid before writing a single agent profile:

**Trap 1: One mega-agent.** "I'll just have one Claude Code agent that does everything." The result: skills don't trigger reliably (descriptions overlap), the agent's Environment block balloons to dozens of credentials (broad blast radius on any leak), system prompt becomes a kitchen sink. You lose the per-task focus that makes agents useful.

**Trap 2: One agent per project.** "I'll have a separate agent for each app." That agent has to context-switch between frontend, backend, deploy, marketing for that app. Same monolithic problem, just with N copies. Multica's quickstart explicitly recommends *role* separation: `Builder`, `Reviewer`, `Ops` — applied across all your projects via Workspace Context.

**Trap 3: One agent per task type, taken to absurdity.** "I need an agent for each TypeScript file pattern." You're now writing routing logic by hand instead of using the orchestrator-workers pattern.

The right granularity sits between these: a role is **a stable profile of skills + API access + tone + judgment** that recurs across projects. "Backend Builder" is a role. "ProjectX-API-Builder" is not.

---

## 2. How many agents is right *for you*

A 4-question diagnostic. Pick the row that fits.

| Stage | Example | Agents | Daily active |
|---|---|---|---|
| **Solo, pre-revenue, building MVP** | First 3 months on a new SaaS | 6–8 | 2–3 |
| **Solo, post-launch, growing** | First customers, content engine starting | 10–14 | 3–5 |
| **Solo + 1–2 contractors / co-founder** | Real GTM motion, customer support load | 14–20 | 5+ |
| **5+ humans + agents** | Small team where agents are second-class teammates | 20+ | 10+ |
| **Agency / multi-product studio** | One workspace per client; reuse the roster | 12 × N workspaces | varies |

The ceiling isn't tokens or compute — it's *your* triage capacity. Each running agent comments on issues, raises blockers, and asks for review. If you can't keep up with their comments, the team is too big regardless of how cheap inference gets.

A useful rule: assume each active agent generates ~5–15 minutes of human review per hour of agent runtime. A 5-agent active set running for 4 hours generates roughly a half-day of human triage. That's the real budget.

---

## 3. Coverage map — what a "complete" team needs to cover

Before listing roles, here are the **functions** a solo founder running a SaaS + GTM motion has to cover. Each needs at least one agent (sometimes shared, sometimes split).

| Domain | Sub-functions |
|---|---|
| **Coordination** | Backlog grooming · roadmap · standups · reporting · cross-agent dispatch |
| **Engineering** | Architecture · frontend · backend · code review · testing · deploy/ops · data |
| **Design** | Brand & visual · UX flows · microcopy · iconography |
| **Research** | Customer/market discovery · technical evaluation · competitive intel |
| **Content & SEO** | Long-form articles · technical docs · landing-page copy · keyword research · on-page SEO |
| **Social media** | Daily posts · threads · video scripts · scheduling · engagement |
| **Creative** | Ad copy · headlines · email subject lines · visuals · video |
| **Lifecycle marketing** | Onboarding emails · drip campaigns · transactional · reactivation |
| **Growth & analytics** | Funnel analysis · A/B tests · attribution · cohort reports |
| **Sales** | Lead research · cold outreach · CRM hygiene · pricing experiments |
| **Customer success** | Ticket triage · KB authoring · Loom-equivalent answers · onboarding |
| **Back-office** | Legal/TOS/privacy · finance/runway · vendor mgmt · compliance |

Twelve domains, ~30 sub-functions. The roster below collapses these to ~18 agents (some agents own multiple sub-functions; some sub-functions are owned by Autopilots, not standing agents).

---

## 4. The roster — 18 agents, 6 tiers

> Tier numbers below are **hiring order**, not "importance". Tier 1 = configure on day one. Tier 6 = only when the specific need is real.

### Tier 1 — Day one (6 agents)

| # | Agent | Why first |
|---|---|---|
| 1 | **Chief of Staff** (PM/coordinator) | The meta-agent; runs daily standup Autopilot, grooms backlog, maintains roadmap. Without it the team has no rhythm. |
| 2 | **Backend Builder** | API, DB, auth, business logic. The hot-path of a SaaS. |
| 3 | **Frontend Builder** | UI components, pages, design-system glue. |
| 4 | **Reviewer** | Independent code review on every PR; security check; "second eye" enforcement. |
| 5 | **Researcher** (general) | Customer discovery, competitive intel, technical evaluations. One agent covers all three at this stage. |
| 6 | **Content Writer** | Landing-page copy, README, launch posts, basic SEO articles. The writing voice that ships externally before you have a marketing engine. |

### Tier 2 — Post-MVP, pre-traction (+5 = 11 agents)

| # | Agent | When to add |
|---|---|---|
| 7 | **DevOps / SRE** | Once you have a real deploy pipeline and can break prod. |
| 8 | **QA / Tester** | When manual testing becomes the bottleneck (typically post-MVP). |
| 9 | **Designer** (brand + visual) | Before public launch — brand identity, design system, marketing visuals. |
| 10 | **Social Media Manager** | When you start daily/weekly posting cadence. |
| 11 | **Customer Support** | First real users; ticket triage and KB authoring. |

### Tier 3 — Scaling GTM (+3 = 14 agents)

| # | Agent | When to add |
|---|---|---|
| 12 | **SEO Specialist** | When content volume > 5 posts/month and you want compounding traffic. |
| 13 | **Growth / Analytics** | When you have funnel data worth analyzing (~1k MAU). |
| 14 | **Lifecycle / Email** | When transactional + onboarding emails need ownership beyond "Backend Builder hardcoded a template". |

### Tier 4 — Specialist creative (+2 = 16 agents)

| # | Agent | When to add |
|---|---|---|
| 15 | **Copywriter** (ads/headlines/cold) | When you start paid acquisition or cold outreach. |
| 16 | **Visual Producer** | When social/ads need real images/video, not stock. |

### Tier 5 — Sales motion (+1 = 17 agents)

| # | Agent | When to add |
|---|---|---|
| 17 | **Sales / Outreach** | When you have ICP clarity and a real sales motion (vs. self-serve). |

### Tier 6 — Back-office (+1 = 18 agents)

| # | Agent | When to add |
|---|---|---|
| 18 | **Compliance / Legal Reader** | When you handle real PII, take payments, sign vendor MSAs. |

**What's deliberately not on the list**: per-language, per-framework, or per-product agents. Use Workspace Context + skills to scope behavior. Cloning the same role 5 times for 5 stacks is the anti-pattern §1 warned about.

---

## 5. Agent specs

Each spec follows the **4-part brief** standard from `agent-orchestration.md` §4.1: objective, output format, tool guidance, task boundaries. Plus the Multica-specific levers: API credentials (set in the agent's Environment field), attached skills, hiring trigger.

> **Note on tooling**: this roster uses **direct API calls** (CLIs and HTTP) over MCP. Multica's `agent.mcp_config` column exists but the GUI tab is still in flight ([PR #1221](https://github.com/multica-ai/multica/pull/1221)) and Codex injection (PR #1288) is open. Every external service this roster touches has a CLI (`gh`, `supabase`, `stripe`, `multica`, `vercel`, `npx playwright`) or a documented HTTP API the agent can drive from Bash. Credentials live in each agent's Environment field. When the MCP UI ships, this roster can layer MCPs on top of the same env-var set without rewriting agent profiles.

> **Reading note**: "Skills" cites entries from `skills-bank.md`. Stars indicate skills the user already has installed locally per `skills-bank.md` §8.

---

### 5.1 Chief of Staff

**Role**: Multica's quarterback. Owns backlog hygiene, weekly roadmap review, daily standup Autopilot, dispatch decisions ("which agent should pick this up?").

**Primary deliverables**: weekly progress digest, monthly roadmap update, fresh issue triage, agent escalation routing.

**System prompt seed**:
> You are the chief of staff for a one-human startup. You don't write product code. You triage incoming issues, group related work into projects, decide which agent should own each issue (by reading their attached skills), maintain the roadmap, and send daily/weekly digests. When unsure, ask the human; never assign without confidence.

**APIs / tools**: Multica REST API + `multica` CLI (issue/agent CRUD) · `gh` CLI (PR/issue link) · Slack Web API (`chat.postMessage` to a draft channel) · Google Calendar API (deadline awareness, read-only).

**Skills**: `gsd-progress`-style workflow skills if available · `internal-comms` (digest writing) · `skill-creator` (so it can author new skills when it identifies a gap).

**Autopilots it owns**:
- Daily 9am — *standup* (`run_only`): scan yesterday's activity, summarize, post to Slack/Inbox.
- Monday 8am — *weekly digest*: KPI snapshot, blockers, what's shipping this week.
- Friday 5pm — *backlog grooming*: stale issues, missing assignees, dupe detection.

**Anti-scope**: never edits source code; never closes issues without human acknowledgment; never auto-merges PRs.

**Hiring trigger**: day one. Without it, the rest of the team has no dispatcher.

---

### 5.2 Backend Builder

**Role**: API, database, auth, business logic, server-side integrations.

**Primary deliverables**: endpoints, migrations, services, tests for the above, scoped PRs.

**System prompt seed**:
> You are the backend engineer. Your job is to extend the API and data model in service of the issue assigned to you. Follow the workspace `CLAUDE.md` for stack conventions. Always write a migration before modifying schema. Always add a test before claiming the issue is done. Open a PR; never push to main.

**APIs / tools**: `gh` CLI · `supabase` CLI + read-only Postgres role (gate writes to migration files) · Sentry REST API (read errors when debugging) · workspace filesystem · WebFetch on official docs sites.

**Skills**: `claude-api` ★ · stack-native skills (`supabase:supabase` ★, `vercel:vercel-functions` ★, `vercel:vercel-storage` ★) · `stripe/stripe-best-practices` if Stripe is in scope · `better-auth/*` if relevant · `webapp-testing` ★ · `simplify` ★.

**Anti-scope**: no UI components, no marketing copy, no infra changes (DevOps owns those), no design tokens.

**Hiring trigger**: day one.

---

### 5.3 Frontend Builder

**Role**: UI components, pages, design-system glue, client-side state, accessibility.

**Primary deliverables**: components, page routes, e2e-testable flows, Storybook entries (if used), Lighthouse-passing pages.

**System prompt seed**:
> You are the frontend engineer. Build distinctive, accessible UIs — not generic AI-slop. Commit to a deliberate aesthetic before coding (per `frontend-design`). Use the workspace's design tokens. Every interactive flow needs at least one Playwright test. Open a PR; never push to main.

**APIs / tools**: `gh` CLI · `vercel` CLI (read-only deployment metadata) · `npx playwright` (visual checks) · workspace filesystem · WebFetch on official docs · Figma REST API (read-only Dev Mode, when a designer agent produces specs).

**Skills**: `frontend-design` ★ · `ui-ux-pro-max` ★ · `refactoring-ui` ★ · `tailwind-v4-shadcn` ★ · `shadcn` ★ · `vercel:react-best-practices` ★ · `vercel:nextjs` ★ · `accessibility-a11y` ★ · `webapp-testing`.

**Anti-scope**: no API or DB changes, no infrastructure, no copy decisions beyond microcopy (that's the UX writer's call when one exists).

**Hiring trigger**: day one.

---

### 5.4 Reviewer

**Role**: Independent code review on every PR. Catches bugs, security issues, style drift. Pairs naturally with `evaluator-optimizer`: Builder generates, Reviewer scores.

**Primary deliverables**: PR reviews with line-level comments, security flags, regression risks, approve/request-changes verdict.

**System prompt seed**:
> You are an independent reviewer. You did not write this code. Read the diff and the issue, then comment on bugs, missing tests, security risks, and style drift versus the workspace conventions. Approve only if you would ship it yourself. Never push commits — comments only.

**APIs / tools**: `gh` CLI (read-only PR access + review comments) · Sentry REST API (recent errors in touched paths) · workspace filesystem (read-only).

**Skills**: `review` ★ · `security-review` ★ · `vercel:react-best-practices` ★ for TSX-heavy reviews · `simplify` ★ · stack-native review skills.

**Different model from Builder**: this is an `evaluator-optimizer` setup. Run Reviewer on a different model (e.g., Codex if Builder is Claude Code) to get adversarial diversity per `agent-orchestration.md` §5 anti-drift mitigation.

**Anti-scope**: no commits; no opening of new PRs; no autoformatting suggestions Builder's tools already enforce.

**Hiring trigger**: day one (the cost of a missed bug grows fast).

**Autopilot**: webhook on PR-open → run Reviewer.

---

### 5.5 Researcher (general)

**Role**: Customer discovery, competitive intel, technical evaluations. At Tier 1 stage, one agent covers all three; later it splits into Market Researcher + Technical Researcher.

**Primary deliverables**: structured research reports (markdown with citations), competitive teardown tables, library evaluation matrices, customer-interview synthesis.

**System prompt seed**:
> You are the research lead. Output structured reports with citations. Prefer primary sources (docs, GitHub, original research) over SEO content. Always include a "what we don't know" section. Never invent statistics; if a claim has no source, say so.

**APIs / tools**: Exa REST API (semantic search; free tier) · Tavily REST API (keyword backup) · Firecrawl REST API (deep web scraping) · `gh` CLI (read repos) · Notion REST API (write drafts only) · WebFetch on official docs.

**Skills**: ad-hoc — no canonical research skill yet. Build one with `skill-creator` that encodes your house rubric (5-axis from `agent-orchestration.md` §4.7).

**Anti-scope**: no code changes, no marketing content (that's the Content Writer's voice), no roadmap decisions (that's the human + Chief of Staff).

**Hiring trigger**: day one. Even at MVP stage, you need someone evaluating libraries and reading the competitive landscape.

**Cost note**: Research benefits most from multi-agent (`agent-orchestration.md` §8 — the +90.2% number was on research workloads). When you grow this into a "research squad," it's the canonical orchestrator-workers use case.

---

### 5.6 Content Writer

**Role**: Long-form articles, landing-page copy, README/changelog, launch announcements, technical docs. Owns "the voice" externally until you split into specialist marketing roles.

**Primary deliverables**: 1500–3000-word articles, landing-page sections, tweet-thread companions to articles, doc updates.

**System prompt seed**:
> You are the content writer. Write in the workspace's brand voice (see Workspace Context). One distinct claim per paragraph. Skim-friendly headings. Cite sources for any factual claim. Never write SEO-stuffed copy; write what you would actually want to read.

**APIs / tools**: Notion REST API or Google Drive API (drafts only) · `gh` CLI (for technical content tied to commits, PR-only no-merge) · Ghost / WordPress / Sanity REST APIs if blog is hosted externally (draft-create only) · Exa or Brave search · Firecrawl (read source articles).

**Skills**: `internal-comms` (Anthropic) · `doc-coauthoring` · Corey Haines marketing skills (Tier 5 in skills bank) · custom voice skill authored via `skill-creator`.

**Anti-scope**: no code, no design, no scheduling/distribution (Social Media owns that), no email lifecycle (Lifecycle agent owns that once it exists).

**Hiring trigger**: day one. The first thing your SaaS needs externally is a landing page that doesn't read like ChatGPT.

---

### 5.7 DevOps / SRE *(Tier 2)*

**Role**: Deploy pipelines, infra, monitoring, incident response, env management.

**Primary deliverables**: CI/CD changes, infra-as-code commits, dashboards, on-call runbooks, post-mortems.

**System prompt seed**:
> You are the DevOps engineer. Treat infra changes like surgery: smallest possible diff, behind feature flags where possible, with a rollback plan in the PR description. You have read access to logs and metrics; you do not push code that changes business logic.

**APIs / tools**: `vercel` CLI · GitHub Actions REST API (`gh workflow`) · Sentry REST API · Datadog or Posthog REST API (depending on stack) · AWS/GCP CLIs if you self-host · Slack Web API (incident channel).

**Skills**: `vercel:deploy` ★ · `vercel:deployments-cicd` ★ · `vercel:env` ★ · `vercel:env-vars` ★ · `vercel:status` ★ · `vercel:verification` ★ · Sentry/Datadog skills (Tier 4 in skills bank) · Trail of Bits security skills if compliance-heavy.

**Anti-scope**: no business-logic changes; no UI; no migrations (Backend owns those, but DevOps reviews them for safety).

**Hiring trigger**: when you have a deploy pipeline that can break prod (typically post-MVP).

**Autopilot**: nightly — check error budget, deploy health, dependency vulnerabilities; file issues if anomalies.

---

### 5.8 QA / Tester *(Tier 2)*

**Role**: End-to-end test coverage, regression sweeps, bug reproduction.

**Primary deliverables**: Playwright e2e suites, bug repro scripts, regression reports, flaky-test fixes.

**System prompt seed**:
> You are the QA engineer. Your job is to find bugs that the Builder didn't catch. Write Playwright tests for new flows, run regression sweeps before releases, file repro-able bug reports. You can fix flaky tests; you do not fix production bugs (file an issue, assign Backend or Frontend).

**APIs / tools**: `npx playwright` CLI · `gh` CLI · Sentry REST API (correlate user-reported bugs with stack traces).

**Skills**: `webapp-testing` · `playwright-e2e-testing` ★ · `accessibility-a11y` ★.

**Anti-scope**: no feature work; no production fixes (just bug reports).

**Hiring trigger**: when manual testing becomes a bottleneck.

**Autopilot**: pre-deploy webhook → run smoke suite, comment results on the PR.

---

### 5.9 Designer (brand + visual) *(Tier 2)*

**Role**: Brand identity, design system, marketing visuals, mockups for new UI.

**Primary deliverables**: Figma files (or equivalent), design tokens, marketing graphics, social-media template kits, brand guidelines doc.

**System prompt seed**:
> You are the brand and visual designer. Commit to a clear aesthetic direction before mocking anything. Output design tokens (colors, typography scale, spacing, motion) as code, not just images, so the Frontend Builder can consume them. For marketing graphics, deliver editable source files plus exports.

**APIs / tools**: Figma REST API · workspace filesystem (write SVG/PNG) · `canvas-design` and `algorithmic-art` skills run via Bash · Google Drive / Notion REST API (asset library).

**Skills**: `frontend-design` ★ · `ui-ux-pro-max` ★ · `canvas-design` · `algorithmic-art` · `theme-factory` · `brand-guidelines` (use as template) · Anthropic creative suite generally.

**Anti-scope**: no production code; no marketing copy; no campaign strategy (that's Growth/Content's call).

**Hiring trigger**: before public launch.

---

### 5.10 Social Media Manager *(Tier 2)*

**Role**: Daily/weekly posts, threads, video scripts, engagement, scheduling.

**Primary deliverables**: scheduled posts across X/LinkedIn/Threads/etc., thread drafts, replies to inbound, weekly engagement report.

**System prompt seed**:
> You are the social media manager. Maintain the content calendar (one source of truth — the Multica project board for "social"). Adapt long-form content from Content Writer into platform-native posts. Never post without explicit human approval; draft and queue only.

**APIs / tools**: Buffer or Hypefury REST API (scheduling) · X / LinkedIn REST APIs (read engagement) · Multica REST API + `multica` CLI (project board for content calendar) · Google Drive API (asset library from Designer).

**Skills**: Corey Haines marketing skill set (Tier 5) · custom platform-style skill per platform (X-thread voice ≠ LinkedIn voice).

**Anti-scope**: no autonomous posting (queue + human approve); no paid-ad copy (that's Copywriter); no email (that's Lifecycle).

**Hiring trigger**: when you commit to a posting cadence.

**Autopilot**: Monday 8am — propose the week's posts based on the content calendar + recent shipping; human approves Monday morning, posts go out through the week.

---

### 5.11 Customer Support *(Tier 2)*

**Role**: Inbound ticket triage, FAQ/KB authoring, draft responses.

**Primary deliverables**: ticket categorization, draft responses (human-approved before send), KB articles, weekly "top issues" report.

**System prompt seed**:
> You are the support agent. For every inbound ticket: categorize, search the KB, draft a response, flag if it requires engineering. Never auto-send; always queue for human approval. If a question is asked >3 times across tickets, file an issue to add it to the KB.

**APIs / tools**: Help-desk REST API (Intercom / HelpScout / Plain / Zendesk) · Multica REST API + `multica` CLI (file engineering issues) · Sentry REST API (correlate with errors) · Google Drive / Notion REST API (KB).

**Skills**: `internal-comms` for tone · workspace-specific support tone skill · `gsd-add-todo`-style skills for filing follow-ups.

**Anti-scope**: no autonomous sending; no production access; no refund/billing actions without human approval.

**Hiring trigger**: first paying customers / first wave of inbound questions.

**Autopilot**: hourly — pull new tickets, draft responses; human reviews queue once a day.

---

### 5.12 SEO Specialist *(Tier 3)*

**Role**: Keyword research, on-page SEO, technical SEO audits, content briefs.

**Primary deliverables**: keyword target lists with intent classification, content briefs handed to Content Writer, on-page SEO checklists per published article, monthly technical audit (Lighthouse, schema, sitemap).

**System prompt seed**:
> You are the SEO specialist. You do not write articles — you brief the Content Writer. Keyword choices must be defended with data (search volume, difficulty, business relevance). Every brief includes the target query, search intent, competitive set, and angle that's actually defensible.

**APIs / tools**: Ahrefs / Semrush / DataForSEO REST APIs · Google Search Console REST API · Lighthouse CLI (via Playwright) · Firecrawl REST API (competitive page analysis).

**Skills**: Addy Osmani Web Quality (Tier 5) · custom SEO-rubric skill via `skill-creator`.

**Anti-scope**: no writing (briefs only); no code changes (files issues for technical SEO fixes).

**Hiring trigger**: content volume > 5 posts/month.

---

### 5.13 Growth / Analytics *(Tier 3)*

**Role**: Funnel analysis, A/B tests, attribution, cohort reports, KPI ownership.

**Primary deliverables**: weekly KPI dashboard, A/B test designs and post-mortems, attribution model, retention/cohort reports.

**System prompt seed**:
> You are the growth analyst. You define metrics, run A/B tests, and explain numbers honestly. When a test is inconclusive, say so. When metrics drop, file an investigation issue with hypotheses. Don't optimize vanity metrics.

**APIs / tools**: Posthog / Mixpanel / Amplitude REST APIs · Stripe REST API (revenue, read-only) · Postgres / BigQuery via CLI (raw data, read-only) · Notion or Google Drive REST API (reports).

**Skills**: custom analytics rubric skill · `xlsx` ★ for ad-hoc spreadsheet analysis · Tinybird skill if you self-host analytics.

**Anti-scope**: no production code; no design; doesn't run experiments without human approval (writes the design doc, hands off to Frontend/Backend).

**Hiring trigger**: ~1k MAU and a real conversion funnel.

**Autopilot**: Monday 7am — KPI snapshot for the week; alert if any metric drops > X%.

---

### 5.14 Lifecycle / Email *(Tier 3)*

**Role**: Onboarding sequences, drip campaigns, transactional copy, reactivation.

**Primary deliverables**: email sequences (subject, preheader, body, CTA), transactional copy templates, lifecycle audit reports.

**System prompt seed**:
> You are the lifecycle marketer. Every email has a single job and a single CTA. Onboarding emails reference what the user actually did in the product (event-driven, not time-based, when possible). Reactivation must be honest — never dark patterns. Always coordinate with Backend on event payloads.

**APIs / tools**: Resend / Postmark / Loops REST APIs · Customer.io / Braze REST API if used · Stripe REST API (lifecycle on billing events) · Posthog REST API (event triggers).

**Skills**: `internal-comms` · custom email-tone skill · `trycourier` skill if multi-channel.

**Anti-scope**: no autonomous sending to live lists; no acquisition email blasts (that's a different game — and probably the Copywriter).

**Hiring trigger**: when transactional + onboarding emails need ownership beyond a hardcoded template.

---

### 5.15 Copywriter (ads/headlines/cold) *(Tier 4)*

**Role**: Ad copy, paid-search headlines, cold-email opens, A/B variants for high-leverage copy.

**Primary deliverables**: ad sets with variants, cold-email sequences, landing-page hero variants, headline/subhead packs.

**System prompt seed**:
> You are the conversion copywriter. Every line has to earn its place. Lead with the customer's problem, not your feature. Generate 5 variants per asset minimum. Use the brand voice but lean punchier than long-form content.

**APIs / tools**: Google Drive / Notion REST API (drafts) · Meta / Google Ads REST APIs (read-only inspection of running ads) · Posthog REST API (which copy actually converts).

**Skills**: Kim Barrett advertising skill set (Tier 5) · workspace voice skill · A/B-rubric skill via `skill-creator`.

**Anti-scope**: no autonomous spend; no scheduling (Social/Lifecycle owns delivery); no long-form (Content Writer's voice).

**Hiring trigger**: paid acquisition or cold outreach starts.

---

### 5.16 Visual Producer *(Tier 4)*

**Role**: Images, short-form video, thumbnails, ad creatives, animated GIFs.

**Primary deliverables**: post-ready visuals, ad creative bundles, video scripts + edits, thumbnails for blog/YouTube.

**System prompt seed**:
> You are the visual producer. You ship usable assets, not concepts. Every output includes the editable source plus the rendered export. Branded templates first; one-offs only when justified.

**APIs / tools**: fal.ai / Replicate / Hugging Face REST APIs (image and video generation) · workspace filesystem (asset library) · `npx remotion render` (if doing programmatic video) · `gsap` skill for animation specs.

**Skills**: Remotion skill (security-audited; ~117k installs) · `gsap` (GreenSock) · `slack-gif-creator` (Anthropic) · `canvas-design` · `algorithmic-art` · fal.ai/Replicate skills.

**Anti-scope**: no design system changes (Designer owns those); no copy (Copywriter owns those).

**Hiring trigger**: when stock images stop being acceptable on social/ads.

---

### 5.17 Sales / Outreach *(Tier 5)*

**Role**: Lead research, list building, cold outreach sequences, CRM hygiene.

**Primary deliverables**: ICP-matched lead lists with notes, multi-touch outreach sequences, CRM updates per touch, weekly pipeline report.

**System prompt seed**:
> You are sales ops, not a closer. You do research, build lists, draft outreach, and keep the CRM clean. The human handles every actual conversation with a lead. Drafts are queued for approval; nothing sends without it.

**APIs / tools**: Attio / HubSpot / Pipedrive REST APIs · LinkedIn / Apollo REST APIs (lead research) · Resend or Apollo email REST APIs · Firecrawl REST API (company research) · Multica REST API + `multica` CLI (file an issue when a lead requires a custom build).

**Skills**: Garry Tan / startup-stack skills (Tier 5) · custom ICP-rubric skill · `internal-comms`.

**Anti-scope**: no autonomous sending to ICP; no pricing changes; no contract negotiation.

**Hiring trigger**: when self-serve isn't your only motion.

---

### 5.18 Compliance / Legal Reader *(Tier 6)*

**Role**: Read TOS/privacy/MSA drafts, flag GDPR/CCPA/SOC2 gaps, surface vendor risks.

**Primary deliverables**: redlines on contracts and policies, compliance checklists, vendor risk reports.

**System prompt seed**:
> You are a compliance reader. You are not a lawyer; everything you flag is for human review with a real attorney. You read carefully, you cite the relevant law/standard, and you propose specific edits. You never approve a contract — only annotate it.

**APIs / tools**: Google Drive / Notion REST API (where docs live) · Vanta or Drata REST API if SOC2 motion · `gh` CLI (for code-level compliance reviews like data-handling).

**Skills**: Trail of Bits security skills · custom compliance-checklist skill · `pdf` ★ (for redlining PDFs).

**Anti-scope**: no signing, no approving, no public claims about compliance posture.

**Hiring trigger**: real PII handled, real payments processed, first vendor MSA, or SOC2 begins.

---

## 6. Scenario playbooks — orchestration patterns in action

Each scenario shows: (a) which agents fire, (b) which `agent-orchestration.md` pattern is in play, (c) how artifacts hand off (filesystem-based, per §4.3 of the orchestration doc).

---

### 6.1 Ship a SaaS MVP from idea to launch

**Pattern stack**: orchestrator-workers (Chief of Staff decomposes) + parallelization sectioning (Frontend/Backend in parallel) + evaluator-optimizer (Builder + Reviewer per PR).

**Flow**:
1. Human files the seed issue: *"Build a usage-based billing SaaS — landing, signup, dashboard, Stripe metering."*
2. **Chief of Staff** decomposes into a project with 12–20 child issues: domain model, auth flow, dashboard skeleton, billing integration, landing page, etc.
3. **Researcher** writes `RESEARCH.md` artifact in the work_dir on Stripe metering options + competitor pricing.
4. **Designer** mocks landing + dashboard wireframes; commits Figma links + design tokens to the workspace.
5. **Backend Builder** + **Frontend Builder** pick up child issues in parallel — each has a non-overlapping scope (this is the §4.1 task-boundary rule).
6. Every PR fires **Reviewer** via PR-open Autopilot. Builder iterates until Reviewer approves.
7. **Content Writer** drafts landing copy from the Researcher's competitive analysis.
8. **DevOps** sets up Vercel + Stripe webhook handlers + Sentry.
9. **QA** writes Playwright e2e for signup + checkout.
10. Pre-launch: human reviews everything, **Chief of Staff** files the launch checklist as a child project.

**Artifact handoff**: every agent writes to the same Multica `work_dir` per task. Reviewer reads Builder's diff straight from git. Designer's tokens are imported by Frontend at build time. No chat-summary "telephone game".

**Token budget**: this is a multi-week, multi-agent, ~15× single-chat-cost workload. The output is a shipped SaaS — value comfortably exceeds the break-even from `agent-orchestration.md` §8.

---

### 6.2 Run a content & SEO engine

**Pattern stack**: prompt chaining (SEO → Writer → Reviewer → Designer for visuals → Social adapts).

**Flow** (weekly cadence, mostly Autopilot-driven):
1. **Autopilot** Monday 6am — **SEO Specialist** runs keyword scan, files 3 article briefs as issues, assigns to **Content Writer**.
2. **Content Writer** drafts each article (3-day SLA, parallel). Drafts land in Drive.
3. **Reviewer** runs an editorial pass for tone + factual accuracy.
4. **Designer** generates header image + 2 in-article diagrams per article.
5. **Frontend Builder** publishes (if blog is in the same repo) or **Content Writer** creates the CMS draft via the platform's REST API.
6. **Social Media Manager** drafts X thread + LinkedIn carousel + reel script per article — queued for human approval Friday.
7. **Autopilot** Monday — **SEO Specialist** reports last week's traffic per article; flags underperformers.

**Pattern detail**: this is a fixed-stage chain — exactly the `prompt chaining` pattern (`agent-orchestration.md` §3.1). One stage's failure poisons the chain; instrument each artifact's existence as the stage gate.

---

### 6.3 Run a social media presence (without being chronically online)

**Pattern stack**: Autopilot fan-out (one trigger spawns multiple platform tasks) + voting (multiple Copywriter variants for high-leverage posts).

**Flow**:
1. **Autopilot** daily 9am — **Social Media Manager** scans yesterday's product changes (via Multica activity log), drafts that day's post.
2. For high-stakes posts (launch, pricing change, milestone) — **Copywriter** generates 5 variants (`parallelization (voting)` from §3.4), human picks one.
3. **Visual Producer** generates platform-sized graphics (X 16:9, LinkedIn 1.91:1, IG 1:1).
4. Human approves in a daily 5-min triage; Autopilot Buffer-pushes scheduled posts.
5. **Autopilot** weekly — pull engagement, identify which post-types performed; **Growth/Analytics** writes a learnings note.

**Cost note**: social is the cheapest place to multi-agent because each post is small. The 15× token cost is dominated by image generation, not LLM calls.

---

### 6.4 Customer discovery / market research sprint

**Pattern stack**: orchestrator-workers — the canonical Anthropic +90.2% scenario (`agent-orchestration.md` §8).

**Flow** (1 week, time-boxed):
1. Human files issue: *"Should we build feature X? Research and recommend."*
2. **Chief of Staff** decomposes into ≤5 sub-tasks (per §4.2 — 3–5 sub-agents). Examples: competitive teardown, Reddit/HN sentiment scan, pricing-page comparison, customer-interview synthesis, technical feasibility.
3. Each sub-task is a child issue, all assigned to the (single) **Researcher** agent — but each runs as a separate `agent_task_queue` entry with its own `work_dir`. Effectively this is one role doing 5 parallel tasks.
4. Each task writes its own `REPORT-<sub>.md` artifact.
5. **Researcher** runs a final synthesis task that reads all 5 reports and writes `SUMMARY.md` with recommendation.
6. Human reads SUMMARY, decides go/no-go.

**5-axis judge** per `agent-orchestration.md` §4.7 — score the SUMMARY on factual accuracy, citation accuracy, completeness, source quality, tool efficiency. If any axis < 0.6, re-run the weakest sub-task.

---

### 6.5 Indie launch (Product Hunt / HN / X)

**Pattern stack**: parallel sectioning (each platform's collateral is independent) + evaluator (Reviewer pass on every asset before launch).

**Flow** (2 weeks pre-launch):
1. **Chief of Staff** files a launch project with platform-specific child issues.
2. **Content Writer** + **Copywriter** + **Visual Producer** + **Frontend Builder** each pick child issues.
   - Content Writer: launch blog post, "behind the scenes" article, Show HN intro.
   - Copywriter: PH tagline + first comment, X thread, LinkedIn post, email-to-network template.
   - Visual Producer: PH gallery (4 images + 1 video), social cards.
   - Frontend Builder: landing-page launch tweaks (countdown banner, day-of CTA).
3. **Designer** runs a visual-consistency pass across all assets.
4. **Reviewer** runs a copy-consistency pass.
5. **DevOps** preps a deploy-freeze and rollback plan for launch day.
6. T-1 day: human dry-run of the launch checklist, **Chief of Staff** confirms all assets in place.
7. Launch day: **Customer Support** is on high alert; **Growth/Analytics** monitors funnel in real time and pings if anything breaks.

---

### 6.6 Maintain & scale a shipped SaaS

**Pattern stack**: Autopilot-driven steady state, with occasional orchestrator-workers for big initiatives.

**Daily/weekly cadence**:
- Hourly — **Customer Support** drafts ticket responses; human triages once.
- Daily 9am — **Chief of Staff** standup digest.
- Daily — **Reviewer** on every PR (PR-open webhook).
- Weekly Mon 6am — **SEO Specialist** brief + **Social Media Manager** week plan.
- Weekly Mon 7am — **Growth/Analytics** KPI snapshot.
- Nightly — **DevOps** error-budget + dep-vuln check.
- Friday 5pm — **Chief of Staff** backlog grooming.

When a real initiative lands (a new feature, a redesign, a competitor response), it temporarily spins up the §6.1 orchestrator-workers flow on top of the steady state.

---

### 6.7 Multi-product solo studio

If you run 2+ products as a solo founder:
- **One Multica workspace per product** (multi-tenancy boundary per `multica.md` §5).
- **Same agent profiles cloned across workspaces**, with different `workspace.context` (stack, brand voice, customer).
- The same human is the orchestrator across all of them — Chief of Staff agents in each workspace coordinate via per-workspace Autopilots.
- Hire order is the same per workspace, but you can defer Tier 4–6 longer if a product is in maintenance mode.

---

### 6.8 Newsletter / creator business (no SaaS engineering needed)

A useful proof that the roster generalizes:
- **Tier 1** — Chief of Staff, Researcher, Content Writer (skip Backend, Frontend, Reviewer).
- **Tier 2** — Designer, Social Media Manager.
- **Tier 3** — SEO Specialist, Lifecycle/Email.
- **Tier 4** — Visual Producer, Copywriter.

That's a 9-agent roster running a creator business. Same coordination layer, different skill mix.

---

## 7. Anti-patterns (catalog)

A checklist of mistakes that show up in solo-founder rosters. Each maps to a specific failure from `agent-orchestration.md` §5.

| Mistake | Symptom | Fix |
|---|---|---|
| **Two agents with overlapping scope** | Both Backend and Frontend agents touch API contracts; merge conflicts; duplicate work | Add explicit "not yours" line to each system prompt; the Chief of Staff enforces dispatch |
| **One agent with no anti-scope** | Agent answers everything but masters nothing; skills don't trigger reliably | Every agent's system prompt ends with anti-scope. No exceptions |
| **Same model + same prompt for Builder and Reviewer** | Reviewer rubber-stamps Builder; bugs slip | Different model for Reviewer (e.g., Codex when Builder is Claude); plus skills like `security-review` Reviewer-only |
| **Agents posting to live channels without approval** | Embarrassing tweet; a real customer email going out wrong | All "send" actions queue for human approval. Default-off, opt-in per agent |
| **Autopilots running on every commit** | Token bill explodes; agents thrash | Autopilots fire on **state-change events** (PR opened, issue created, schedule), not raw commits. Concurrency = `skip` or `queue`, never `replace` for cron jobs |
| **One mega-agent that owns marketing AND engineering** | Skills don't trigger; tone drifts; Environment block balloons with credentials for unrelated services | Split. The whole point of the roster |
| **No Chief of Staff** | Backlog rots; agents idle on stale issues; you become the dispatcher manually | Configure Chief of Staff first, even before Builder agents |
| **Skills attached to all agents "just in case"** | L1 metadata floods every task; signal-to-noise drops; cost goes up linearly | Skills are scoped per agent in Multica — use that. Default to attaching nothing extra |
| **API credentials duplicated across agents that don't need them** | Every agent's Environment carries the same broad-scope tokens; one breach blast-radius covers the whole team | One credential set per agent, scoped to its role. Reviewer doesn't need a Stripe write key |
| **Chasing the +90.2% number on routine tasks** | Multi-agent for everything; bills 15× higher; quality not better on the simple cases | Multi-agent only when the value of the answer > 15× a single chat. Most daily work is single-agent |

---

## 8. The week-by-week hire order

A concrete 8-week plan from "empty Multica workspace" to "running studio".

| Week | Add | Why this week |
|---|---|---|
| 1 | Chief of Staff, Backend Builder, Frontend Builder | The minimum to ship code with structure |
| 2 | Reviewer, Researcher | Now PRs get a second eye; you have research capacity for the first hard tradeoff |
| 3 | Content Writer | Landing page + first README; you can start building public surface area |
| 4 | DevOps/SRE, QA/Tester | Real CI/CD + e2e before MVP launch |
| 5 | Designer, Social Media Manager | Brand identity + social presence pre-launch |
| 6 | Customer Support | First users land; tickets need ownership |
| 7 | SEO Specialist, Growth/Analytics | Traffic engine + funnel measurement once launched |
| 8 | Lifecycle/Email | Onboarding + retention emails as users compound |
| Later | Copywriter, Visual Producer, Sales/Outreach, Compliance | Add on real demand only |

**At week 8**: 13 agents configured, ~4–5 active on a typical day, daily standup and weekly digest running on Autopilots, one orchestrator-workers initiative open at any time.

That is the realistic shape of "doing everything alone with a team of agents."

---

## 9. Wiring it up in Multica — concrete checklist

For each agent in the roster:

1. **Create the agent** (Settings → Agents → New) with name + avatar matching the role.
2. **Pick a runtime/provider** — Claude Code for most; Codex for Reviewer (model diversity); Cursor or another CLI if a specific tool fits the role better.
3. **Paste the system prompt** from §5 (the seed; expand with your house-style additions).
4. **Attach skills** per §5. Don't over-attach — start narrow.
5. **Set the agent's Environment** field with the per-agent API credentials listed in `agents/<name>/skills.md` (per-agent, not workspace-wide per `multica.md` §5).
6. **Workspace Context** — write your stack + brand voice + non-negotiables once, in `workspace.context`. Every agent inherits it.
7. **Configure Autopilots** per §5 (Chief of Staff, Reviewer, DevOps nightly, etc.).
8. **Set anti-scope as a `(custom_args)` system-prompt suffix** if your CLI supports it; otherwise hard-code into the agent profile.

Per-agent overrides via `MULTICA_<NAME>_*` env vars on the daemon (per `multica.md` §10) let you tune model, args, and binary path without touching agent config.

---

## 10. Cross-references and sources

- `agent-orchestration.md` — patterns, parallel limits, sub-agent brief anatomy, evaluation framework, failure-mode catalog.
- `multica.md` — coordination primitives this roster maps onto (workspace, agent, skill, autopilot, task lifecycle).
- `skills-bank.md` — the curated skill catalog every agent in §5 pulls from. Tiers 1–5 of the bank line up with hiring tiers here only loosely; consult both.
- Anthropic, *Building Effective Agents* — https://www.anthropic.com/research/building-effective-agents
- Anthropic Engineering, *How we built our multi-agent research system* — https://www.anthropic.com/engineering/multi-agent-research-system
- Multica product overview (`docs/product-overview.md` in the repo) — schema, lifecycle, autopilot triggers.
- `multica.md` §13 (vibe-coding solo workflow) — the original 3-agent example (Builder / Reviewer / Ops). This file is what that example grows into.
