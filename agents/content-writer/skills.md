# Skills + MCPs for Content Writer

## Skills

### Already installed locally (just attach)

Per `skills-bank.md` §8 the user's installed stack is engineering- and Vercel-heavy. The genuinely-applicable items for a writer agent are slim — most of the bank's writing-side skills are gaps the user will need to add (next section). Attach only what triggers reliably for prose work; don't bloat L1.

| Skill | Source | skills.sh | clawhub.io | Why |
|---|---|---|---|---|
| `pdf` | `anthropics/skills/pdf` | [skills.sh/anthropics/skills/pdf](https://skills.sh/anthropics/skills/pdf) | unreachable (cert/domain expired May 2026) — fallback: [github.com/anthropics/skills/tree/main/pdf](https://github.com/anthropics/skills/tree/main/pdf) | Reading source PDFs (whitepapers, analyst reports) cited in articles. |
| `xlsx` | `anthropics/skills/xlsx` | [skills.sh/anthropics/skills/xlsx](https://skills.sh/anthropics/skills/xlsx) | unreachable — fallback: [github.com/anthropics/skills/tree/main/xlsx](https://github.com/anthropics/skills/tree/main/xlsx) | Reading data that backs claims (benchmark spreadsheets, survey results). Don't *create* charts here — that's Designer / Visual Producer. |
| `simplify` | bundled with Claude Code | n/a (bundled — no marketplace import) | n/a (bundled) | Built-in self-edit pass to cut filler — good fit for "make this paragraph tighter". |

Do **not** attach the Vercel / Supabase / shadcn / `claude-api` / `frontend-design` / `ui-ux-pro-max` stack. They don't trigger on prose, and their L1 metadata burns ~100 tokens each for nothing. The Content Writer's skill list is deliberately short.

### Needs installation

| Skill | Source | skills.sh | clawhub.io | Why |
|---|---|---|---|---|
| `internal-comms` | `anthropics/skills/internal-comms` | [skills.sh/anthropics/skills/internal-comms](https://skills.sh/anthropics/skills/internal-comms) | unreachable (cert/domain expired May 2026) — fallback: [github.com/anthropics/skills/tree/main/internal-comms](https://github.com/anthropics/skills/tree/main/internal-comms) | Drafts launch announcements, change notices, project updates, FAQs. Adjusts tone and density per audience — the structure layer for launches and CHANGELOGs. |
| `doc-coauthoring` | `anthropics/skills/doc-coauthoring` | [skills.sh/anthropics/skills/doc-coauthoring](https://skills.sh/anthropics/skills/doc-coauthoring) | unreachable — fallback: [github.com/anthropics/skills/tree/main/doc-coauthoring](https://github.com/anthropics/skills/tree/main/doc-coauthoring) | Three-phase gather → refine → reader-test workflow. Built-in editorial process for any 1500–3000-word piece. |
| `docx` | `anthropics/skills/docx` | [skills.sh/anthropics/skills/docx](https://skills.sh/anthropics/skills/docx) | unreachable — fallback: [github.com/anthropics/skills/tree/main/docx](https://github.com/anthropics/skills/tree/main/docx) | When briefs land as Word docs from the human or when external CMSs export drafts as `.docx`. |
| Corey Haines marketing skills (`copywriting`, `marketing-psychology`, `pricing-strategy`, `programmatic-seo`, `social-content`, `marketing-ideas`, `seo-audit`, `product-marketing-context`, ...) | `coreyhaines31/marketingskills` (40 skills total) | publisher index: [skills.sh/coreyhaines31/marketingskills](https://skills.sh/coreyhaines31/marketingskills) — examples: [copywriting](https://skills.sh/coreyhaines31/marketingskills/copywriting) · [marketing-psychology](https://skills.sh/coreyhaines31/marketingskills/marketing-psychology) · [pricing-strategy](https://skills.sh/coreyhaines31/marketingskills/pricing-strategy) · [programmatic-seo](https://skills.sh/coreyhaines31/marketingskills/programmatic-seo) · [seo-audit](https://skills.sh/coreyhaines31/marketingskills/seo-audit) · [social-content](https://skills.sh/coreyhaines31/marketingskills/social-content) · [marketing-ideas](https://skills.sh/coreyhaines31/marketingskills/marketing-ideas) · [product-marketing-context](https://skills.sh/coreyhaines31/marketingskills/product-marketing-context) | unreachable — fallback: [github.com/coreyhaines31/marketingskills](https://github.com/coreyhaines31/marketingskills) | 40 playbooks for SaaS marketing voice — landing-page CRO heuristics, positioning teardowns, copywriting checklists. The closest thing to a published "indie SaaS writer" rubric. Subset only — don't load all 40 if a topical few cover the immediate need (positioning, landing-copy, launch). |

### To author via skill-creator (high priority)

The most-leveraged authoring item by far is `brand-voice-<workspace>`. **It is the difference between a "ChatGPT landing page" and a page that sounds like a human wrote it.** Generic advice ("be conversational", "use active voice") gets you to AI-default output. A workspace voice skill — with banned phrases, real-example paragraphs from your existing writing, and a "good vs slop" comparison set — is the only reliable way to push past that ceiling. Author it once per product; attach to this agent only.

| Skill | Purpose |
|---|---|
| `brand-voice-<workspace>` | **Highest-priority custom skill.** Encodes the workspace's voice with: (a) 3 paragraphs of "good" reference prose from existing copy, (b) 3 paragraphs of "AI-slop" failure modes to avoid, (c) banned-phrase list, (d) sentence-rhythm guidelines, (e) audience persona. Authored once per product; re-evaluated quarterly. Use `skill-creator`'s eval loop to verify the description triggers on prose tasks but doesn't fire on code or research. |
| `content-brief-template` | Repeatable structure for accepting briefs from SEO Specialist or human. Forces every brief to specify: target reader, single defensible claim, success criterion, length target, sources to read, ban-list overrides. Triggers when an issue is opened on the Content Writer with a `brief` label or any `## Brief` heading. |
| `article-checklist` | Pre-handoff checklist run as the last step before the agent says "draft ready for review": every claim has a source link? skim-test passes (only headings + first sentences = coherent gist)? at least one concrete example per major section? CTAs present and not stuffed? word count in the requested band? banned phrases absent? Triggers on phrases like "ready for review" or "final draft". |
| `launch-post-shape` *(optional, after first launch)* | Captures the launch-post structure that worked the first time so subsequent launches don't reinvent it. Author *after* you've shipped one launch you're happy with — don't author it from theory. |

## MCP servers (`agent.mcp_config`)

All write-capable MCPs are scoped **draft-only**: Notion writes only to a designated Drafts database; Ghost / WordPress / Sanity create draft posts and never publish; GitHub opens PRs and never merges. This is enforced both at the system-prompt level and at the MCP-token-permission level (use minimum-scope tokens — read-only where possible, draft-only write scopes where not).

| MCP | Scope | Why |
|---|---|---|
| Notion *(if drafts live there)* — [`@notionhq/notion-mcp-server`](https://www.npmjs.com/package/@notionhq/notion-mcp-server) or hosted [mcp.notion.com](https://mcp.notion.com/mcp) | **Write — drafts database only.** Integration token granted access to a single "Content Drafts" database, no other pages. | Workspace's draft landing place when not in-repo. Hosted server is the maintained path; npm package useful for self-host. |
| Google Drive *(alt to Notion)* — Google Drive MCP via `mcp__claude_ai_Google_Drive__*` (workspace OAuth, already in user's tool list) | **Write — single "Content Drafts" folder.** OAuth scope limited to that folder. | Alternative draft store if the team is Drive-native. Pick **one of** Notion or Drive — not both. |
| GitHub — `@modelcontextprotocol/server-github` | **Write — branch + PR only, never merge.** PAT with `repo` scope but the agent's prompt + Reviewer agent enforce no-merge. | (a) In-repo MDX content (Velite / Content Collections / Fumadocs) ships as a PR. (b) README + CHANGELOG live in repos. (c) Read commits + diffs to write technically accurate launch posts and changelog entries. |
| Ghost MCP *(if blog is Ghost)* — [`MFYDev/ghost-mcp`](https://github.com/MFYDev/ghost-mcp) is the most-maintained community option (alternatives: `mtane0412/ghost-mcp-server`, `siva-sub/ghost-cms-mcp-server`). Ghost has an [open proposal for a first-party MCP](https://forum.ghost.org/t/proposal-first-party-mcp-model-context-protocol-server-for-the-admin-api/62774) — adopt when shipped. | **Draft create only.** Admin API key restricted to draft-post creation; never `published` status. | External CMS draft path. |
| WordPress MCP *(if blog is WP)* — [`WordPress/mcp-adapter`](https://developer.wordpress.org/news/2026/02/from-abilities-to-ai-agents-introducing-the-wordpress-mcp-adapter/) (official, Feb 2026; supersedes `Automattic/wordpress-mcp`). Alternative: [`rnaga/wp-mcp`](https://github.com/rnaga/wp-mcp). | **Draft create only.** Application Password scoped to `edit_posts` but agent never calls publish endpoints. | External CMS draft path. |
| Sanity MCP *(if blog is Sanity)* — [official Sanity MCP server](https://www.sanity.io/docs/ai/mcp-server) | **Draft create only — `drafts.*` documents, never publish action.** | External CMS draft path; schema-aware so generated drafts respect content model. |
| Web search — Brave / Exa / similar | **Read-only.** | Source articles, primary references for citations. |
| [Firecrawl](https://github.com/firecrawl/firecrawl-mcp-server) — `npx -y firecrawl-mcp` | **Read-only.** API key only. | Full-page scrape of source articles, competitor landing pages, primary research. The Researcher agent has this too; the Content Writer needs it for inline citation-fetching while drafting. |

**Recommendation for the user's stack** (Vercel + Supabase + Next.js per `skills-bank.md` §8): default to **in-repo MDX** with Velite or Content Collections (Contentlayer is deprecated as of 2024). Blog files live in the same Next.js app, deploy on Vercel preview/prod with the rest of the codebase. MCP set: **GitHub (PR/branch, no-merge) + Notion (drafts) + Firecrawl + web search**. Skip Ghost / WordPress / Sanity unless the blog migrates externally. If a hybrid is needed (engineering posts in-repo, marketing posts in a CMS), attach both — keep both draft-only.

### How to wire this in Multica (no MCP UI page yet)

The cleaned, spec-valid config lives at `agents/content-writer/mcp.json` (sibling file). Multica's agent-settings GUI does not expose an MCP page today — [PR #1221](https://github.com/multica-ai/multica/pull/1221) is still open as of May 2026. The daemon DOES however read `agent.mcp_config` from the DB and pass it to Claude Code via `--mcp-config <tempfile>` (PR #1168, shipped v0.2.6). So:

1. **Load `mcp.json` into the agent's `mcp_config` column** — via REST API, `multica` CLI, or direct SQL. Recipes in `agents/MCP-WIRING.md` Part A2. Custom arguments cannot inject `--mcp-config` (the daemon blocks it via `claudeBlockedArgs`).

2. **In the Multica GUI** (Settings → Agents → Content Writer):
   - **Custom arguments**: leave empty.
   - **Environment** (one `KEY=VALUE` per line — secrets used as `${VAR}` in `mcp.json`):
     ```
     GITHUB_PAT_CONTENT_WRITER=ghp_...
     NOTION_DRAFTS_TOKEN=secret_...
     FIRECRAWL_API_KEY=fc-...
     EXA_API_KEY=...
     ```

Rationale notes that previously lived in narrative around the JSON:
- **github**: PAT scoped to `contents:write`, `pull_requests:write` (no merge), `issues:read`. Branch protection on `main` blocks merges from this PAT — the no-merge rule is enforced by both the PAT scope and the Reviewer agent's gating, defense in depth.
- **notion**: integration token granted access to a single "Content Drafts" database. The agent literally cannot write anywhere else in the workspace.
- **firecrawl** and **exa** are read-only by API design (search + scrape, no write surface).

## Coherence check

The brand-voice skill is what the agent *sounds like*; the draft-only MCPs are what the agent *can do*; the anti-publish system prompt is what the agent *won't try to do even if asked*. Together they produce shippable copy that lands in a human-reviewable state every time — distinctive enough to avoid AI-slop, scoped tightly enough that a misfire cannot publish to production. Skills bias output quality up; MCP scopes cap blast radius down; the system prompt closes the gap between them. Remove any one of the three and the agent either becomes generic (no voice skill), risky (publish-capable MCPs), or aspirationally autonomous (no anti-publish prompt).
