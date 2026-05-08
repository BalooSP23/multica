# Skills + tools for Content Writer

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

The most-leveraged authoring item by far is `brand-voice-<workspace>`. **It is the difference between a "ChatGPT landing page" and a page that sounds like a human wrote it.**

| Skill | Purpose |
|---|---|
| `brand-voice-<workspace>` | **Highest-priority custom skill.** Encodes the workspace's voice with: (a) 3 paragraphs of "good" reference prose from existing copy, (b) 3 paragraphs of "AI-slop" failure modes to avoid, (c) banned-phrase list, (d) sentence-rhythm guidelines, (e) audience persona. Authored once per product; re-evaluated quarterly. Use `skill-creator`'s eval loop to verify the description triggers on prose tasks but doesn't fire on code or research. |
| `content-brief-template` | Repeatable structure for accepting briefs from SEO Specialist or human. Forces every brief to specify: target reader, single defensible claim, success criterion, length target, sources to read, ban-list overrides. |
| `article-checklist` | Pre-handoff checklist run as the last step before "draft ready for review": every claim has a source link? skim-test passes? at least one concrete example per major section? CTAs present? word count in band? banned phrases absent? |
| `launch-post-shape` *(after first launch)* | Captures the launch-post structure that worked. Author *after* you've shipped one launch you're happy with — don't author it from theory. |

**Skills bundled with Claude Code** (no marketplace import): `simplify`. Built-in self-edit pass to cut filler.

## Tools & API access

All write paths are **draft-only**: Notion writes only to a designated Drafts database; Ghost / WordPress / Sanity create draft posts and never publish; GitHub opens PRs and never merges. Enforced both at the system-prompt level and at the API-token-permission level.

| Service | How the agent uses it | Required env vars | Scope |
|---|---|---|---|
| **GitHub** | `gh` CLI for branches and PRs containing in-repo MDX content (`gh pr create`), README + CHANGELOG updates. **Never** `gh pr merge`. Reading commits/diffs to write technically accurate launch posts. | `GH_TOKEN=$GITHUB_PAT_CONTENT_WRITER` (`contents:write`, `pull_requests:write` no-merge, `issues:read`) | No merge. Branch protection on `main` + Reviewer agent gate. |
| **Notion** | Notion REST API: `curl https://api.notion.com/v1/pages` with `Notion-Version: 2022-06-28` to create draft pages under the workspace's "Content Drafts" database. | `NOTION_TOKEN=secret_…` (integration token granted access to one database only) | Scoped write — drafts database only. |
| **Google Drive** *(alt to Notion)* | Drive REST API: `curl https://www.googleapis.com/drive/v3/files` to create files in the workspace's "Content Drafts" folder. OAuth-completed at agent setup. | `GDRIVE_REFRESH_TOKEN` (scope: `drive.file` on the drafts folder only) | Scoped write. Pick **one of** Notion or Drive — not both. |
| **Ghost** *(if blog is Ghost)* | Ghost Admin API: `curl https://<site>.ghost.io/ghost/api/admin/posts/` with `Authorization: Ghost <jwt>` to create `status: 'draft'` posts only. | `GHOST_ADMIN_API_KEY` (Admin API key restricted to Author role; Author role cannot publish — only Editor+ can) | Draft create only. |
| **WordPress** *(if blog is WP)* | WP REST API: `curl https://<site>/wp-json/wp/v2/posts` with `Authorization: Basic <user:app-password>`, body `{"status":"draft"}`. | `WP_USER` · `WP_APP_PASSWORD` (Application Password scoped to a user with `edit_posts` capability only — no `publish_posts`) | Draft create only. |
| **Sanity** *(if blog is Sanity)* | Sanity HTTP API: `curl https://<project>.api.sanity.io/v2024-01-01/data/mutate/<dataset>` with `Authorization: Bearer <token>` and `_id: "drafts.<id>"` documents only. | `SANITY_TOKEN` (Editor role; mutations to `drafts.*` IDs only — never publish action) | Draft create only — `drafts.*` documents. |
| **Web search — Exa** | `curl https://api.exa.ai/search` for source articles, primary references for citations. | `EXA_API_KEY` | Read-only. |
| **Firecrawl** | `curl https://api.firecrawl.dev/v1/scrape` for full-page scrape of source articles, competitor landing pages, primary research. | `FIRECRAWL_API_KEY` (`fc-…`) | Read-only. |
| **Library docs** | Claude Code's WebFetch tool against the relevant docs site for technical-content fact-checking. | none | n/a |

**Recommendation for the user's stack** (Vercel + Supabase + Next.js): default to **in-repo MDX** with Velite or Content Collections (Contentlayer is deprecated as of 2024). Blog files live in the same Next.js app, deploy on Vercel preview/prod with the rest of the codebase. Tool set: **`gh` (PR/branch, no-merge) + Notion REST (drafts) + Firecrawl + Exa**. Skip Ghost / WordPress / Sanity unless the blog migrates externally.

### Wiring in Multica

In the Multica GUI (Settings → Agents → Content Writer), set:

- **Custom arguments**: leave empty.
- **Environment** (one `KEY=VALUE` per line):
  ```
  GH_TOKEN=ghp_...
  NOTION_TOKEN=secret_...
  FIRECRAWL_API_KEY=fc-...
  EXA_API_KEY=...
  ```

The agent invokes `gh`, `curl` from Bash. No MCP server config to maintain.

Rationale notes:
- **github**: PAT scoped to `contents:write`, `pull_requests:write` (no merge), `issues:read`. Branch protection on `main` blocks merges; Reviewer agent's gating is defense in depth.
- **notion**: integration token granted access to a single "Content Drafts" database. The agent literally cannot write anywhere else in the workspace.
- **firecrawl** and **exa** are read-only by API design.

## Coherence check

The brand-voice skill is what the agent *sounds like*; the draft-only API surfaces are what the agent *can do*; the anti-publish system prompt is what the agent *won't try to do even if asked*. Together they produce shippable copy that lands in a human-reviewable state every time — distinctive enough to avoid AI-slop, scoped tightly enough that a misfire cannot publish to production.
