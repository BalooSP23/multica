# Skills + MCPs for Content Writer

## Skills (Multica → `agent_skill` table)

The Content Writer's skill list is deliberately short. Don't pre-attach the Vercel / Supabase / shadcn / `claude-api` / `frontend-design` / `ui-ux-pro-max` stack — they don't trigger on prose, and their L1 metadata burns ~100 tokens each for nothing.

### To attach (import from skills.sh)

| Skill | Why for this role | skills.sh | GitHub fallback |
|---|---|---|---|
| `pdf` | Reading source PDFs (whitepapers, analyst reports) cited in articles. | [skills.sh/anthropics/skills/pdf](https://skills.sh/anthropics/skills/pdf) | [github.com/anthropics/skills/tree/main/pdf](https://github.com/anthropics/skills/tree/main/pdf) |
| `xlsx` | Reading data that backs claims (benchmark spreadsheets, survey results). Don't *create* charts here — that's Designer / Visual Producer. | [skills.sh/anthropics/skills/xlsx](https://skills.sh/anthropics/skills/xlsx) | [github.com/anthropics/skills/tree/main/xlsx](https://github.com/anthropics/skills/tree/main/xlsx) |
| `internal-comms` | Drafts launch announcements, change notices, project updates, FAQs. The structure layer for launches and CHANGELOGs. | [skills.sh/anthropics/skills/internal-comms](https://skills.sh/anthropics/skills/internal-comms) | [github.com/anthropics/skills/tree/main/internal-comms](https://github.com/anthropics/skills/tree/main/internal-comms) |
| `doc-coauthoring` | Three-phase gather → refine → reader-test workflow. Built-in editorial process for any 1500–3000-word piece. | [skills.sh/anthropics/skills/doc-coauthoring](https://skills.sh/anthropics/skills/doc-coauthoring) | [github.com/anthropics/skills/tree/main/doc-coauthoring](https://github.com/anthropics/skills/tree/main/doc-coauthoring) |
| `docx` | When briefs land as Word docs from the human or external CMSs export drafts as `.docx`. | [skills.sh/anthropics/skills/docx](https://skills.sh/anthropics/skills/docx) | [github.com/anthropics/skills/tree/main/docx](https://github.com/anthropics/skills/tree/main/docx) |
| `copywriting` (Corey Haines) | Marketing-voice copywriting checklists. | [skills.sh/coreyhaines31/marketingskills/copywriting](https://skills.sh/coreyhaines31/marketingskills/copywriting) | [github.com/coreyhaines31/marketingskills](https://github.com/coreyhaines31/marketingskills) |
| `marketing-psychology` | Persuasion principles for landing-page / launch-post copy. | [skills.sh/coreyhaines31/marketingskills/marketing-psychology](https://skills.sh/coreyhaines31/marketingskills/marketing-psychology) | [github.com/coreyhaines31/marketingskills](https://github.com/coreyhaines31/marketingskills) |
| `pricing-strategy` | When pricing-page copy is in scope. | [skills.sh/coreyhaines31/marketingskills/pricing-strategy](https://skills.sh/coreyhaines31/marketingskills/pricing-strategy) | [github.com/coreyhaines31/marketingskills](https://github.com/coreyhaines31/marketingskills) |
| `programmatic-seo` | Long-tail page generation when the SEO Specialist (Tier 3) hands off keyword sets. | [skills.sh/coreyhaines31/marketingskills/programmatic-seo](https://skills.sh/coreyhaines31/marketingskills/programmatic-seo) | [github.com/coreyhaines31/marketingskills](https://github.com/coreyhaines31/marketingskills) |
| `seo-audit` | On-page SEO checklist applied to drafts before handoff. | [skills.sh/coreyhaines31/marketingskills/seo-audit](https://skills.sh/coreyhaines31/marketingskills/seo-audit) | [github.com/coreyhaines31/marketingskills](https://github.com/coreyhaines31/marketingskills) |
| `social-content` | Adapting articles into thread/post companions. | [skills.sh/coreyhaines31/marketingskills/social-content](https://skills.sh/coreyhaines31/marketingskills/social-content) | [github.com/coreyhaines31/marketingskills](https://github.com/coreyhaines31/marketingskills) |
| `marketing-ideas` | Brainstorming prompt for content-calendar gaps. | [skills.sh/coreyhaines31/marketingskills/marketing-ideas](https://skills.sh/coreyhaines31/marketingskills/marketing-ideas) | [github.com/coreyhaines31/marketingskills](https://github.com/coreyhaines31/marketingskills) |
| `product-marketing-context` | Positioning teardowns. | [skills.sh/coreyhaines31/marketingskills/product-marketing-context](https://skills.sh/coreyhaines31/marketingskills/product-marketing-context) | [github.com/coreyhaines31/marketingskills](https://github.com/coreyhaines31/marketingskills) |

> The Corey Haines pack has 40 skills total. The 8 above are the highest-leverage subset for SaaS prose. Don't attach all 40 — L1 metadata cost compounds.

### To author via `skill-creator` (highest priority)

The most-leveraged authoring item by far is `brand-voice-<workspace>`. **It is the difference between a "ChatGPT landing page" and a page that sounds like a human wrote it.** Generic advice ("be conversational", "use active voice") gets you to AI-default output. A workspace voice skill — with banned phrases, real-example paragraphs from your existing writing, and a "good vs slop" comparison set — is the only reliable way to push past that ceiling. Author it once per product; attach to this agent only.

| Skill | Purpose |
|---|---|
| `brand-voice-<workspace>` | **Highest-priority custom skill.** Encodes the workspace's voice with: (a) 3 paragraphs of "good" reference prose from existing copy, (b) 3 paragraphs of "AI-slop" failure modes to avoid, (c) banned-phrase list, (d) sentence-rhythm guidelines, (e) audience persona. Authored once per product; re-evaluated quarterly. Use `skill-creator`'s eval loop to verify the description triggers on prose tasks but doesn't fire on code or research. |
| `content-brief-template` | Repeatable structure for accepting briefs from SEO Specialist or human. Forces every brief to specify: target reader, single defensible claim, success criterion, length target, sources to read, ban-list overrides. |
| `article-checklist` | Pre-handoff checklist run as the last step before "draft ready for review": every claim has a source link? skim-test passes? at least one concrete example per major section? CTAs present? word count in band? banned phrases absent? |
| `launch-post-shape` *(after first launch)* | Captures the launch-post structure that worked. Author *after* you've shipped one launch you're happy with — don't author it from theory. |

**Skills bundled with Claude Code** (no marketplace import): `simplify`. Built-in self-edit pass to cut filler — good fit for "make this paragraph tighter".

## MCP servers (`agent.mcp_config`)

All write-capable MCPs are scoped **draft-only**: Notion writes only to a designated Drafts database; Ghost / WordPress / Sanity create draft posts and never publish; GitHub opens PRs and never merges. Enforced both at the system-prompt level and at the MCP-token-permission level.

| MCP | Scope | Why |
|---|---|---|
| Notion (`@notionhq/notion-mcp-server` or hosted [mcp.notion.com](https://mcp.notion.com/mcp)) | **Write — drafts database only.** Integration token granted access to a single "Content Drafts" database, no other pages. | Workspace's draft landing place when not in-repo. Hosted server is the maintained path. |
| Google Drive (workspace OAuth) | **Write — single "Content Drafts" folder.** OAuth scope limited to that folder. | Alternative draft store if the team is Drive-native. Pick **one of** Notion or Drive — not both. |
| GitHub (`@modelcontextprotocol/server-github`) | **Write — branch + PR only, never merge.** PAT with `repo` scope but the agent's prompt + Reviewer agent enforce no-merge. | (a) In-repo MDX content (Velite / Content Collections / Fumadocs) ships as a PR. (b) README + CHANGELOG live in repos. (c) Read commits + diffs to write technically accurate launch posts. |
| Ghost (`MFYDev/ghost-mcp`, if blog is Ghost) | **Draft create only.** Admin API key restricted to draft-post creation; never `published` status. | External CMS draft path. |
| WordPress (`WordPress/mcp-adapter`, official Feb 2026, if blog is WP) | **Draft create only.** Application Password scoped to `edit_posts` but agent never calls publish endpoints. | External CMS draft path. |
| Sanity (official Sanity MCP, if blog is Sanity) | **Draft create only — `drafts.*` documents, never publish action.** | Schema-aware so generated drafts respect content model. |
| Web search — Exa / Brave | **Read-only.** | Source articles, primary references for citations. |
| Firecrawl — `firecrawl-mcp` | **Read-only.** API key only. | Full-page scrape of source articles, competitor landing pages, primary research. |

**Recommendation for the user's stack** (Vercel + Supabase + Next.js): default to **in-repo MDX** with Velite or Content Collections (Contentlayer is deprecated as of 2024). Blog files live in the same Next.js app, deploy on Vercel preview/prod with the rest of the codebase. MCP set: **GitHub (PR/branch, no-merge) + Notion (drafts) + Firecrawl + web search**. Skip Ghost / WordPress / Sanity unless the blog migrates externally.

### How to wire this in Multica

The cleaned, spec-valid config lives at `agents/content-writer/mcp.json`. Multica's agent-settings GUI does not expose an MCP page today — [PR #1221](https://github.com/multica-ai/multica/pull/1221) is open. The daemon reads `agent.mcp_config` from the DB and passes it to Claude Code via `--mcp-config <tempfile>` (PR #1168, shipped v0.2.6). Full procedure in `agents/MCP-WIRING.md` Part A.

In the Multica GUI (Settings → Agents → Content Writer):
- **Custom arguments**: leave empty.
- **Environment** (one `KEY=VALUE` per line — referenced as `${VAR}` in `mcp.json`):
  ```
  GITHUB_PAT_CONTENT_WRITER=ghp_...
  NOTION_DRAFTS_TOKEN=secret_...
  FIRECRAWL_API_KEY=fc-...
  EXA_API_KEY=...
  ```

Rationale notes:
- **github**: PAT scoped to `contents:write`, `pull_requests:write` (no merge), `issues:read`. Branch protection on `main` blocks merges from this PAT. Defense in depth via the Reviewer agent's gating.
- **notion**: integration token granted access to a single "Content Drafts" database. The agent literally cannot write anywhere else in the workspace.
- **firecrawl** and **exa** are read-only by API design.

## Coherence check

The brand-voice skill is what the agent *sounds like*; the draft-only MCPs are what the agent *can do*; the anti-publish system prompt is what the agent *won't try to do even if asked*. Together they produce shippable copy that lands in a human-reviewable state every time — distinctive enough to avoid AI-slop, scoped tightly enough that a misfire cannot publish to production. Skills bias output quality up; MCP scopes cap blast radius down; the system prompt closes the gap between them.
