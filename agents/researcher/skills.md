# Skills + tools for Researcher

## Skills (Multica → `agent_skill` table)

The Researcher is light on skills, heavy on tools — most of the value is "find primary sources and read them carefully," which is web/API work. The skill list focuses on document I/O and the custom rubrics authored via `skill-creator`.

### To attach (import from skills.sh)

| Skill | Why for this role | skills.sh | GitHub fallback |
|---|---|---|---|
| `pdf` | Primary sources are often PDFs (whitepapers, S-1s, vendor security docs, academic research). Without this, the agent skips PDF citations or hallucinates summaries. | [skills.sh/anthropics/skills/pdf](https://skills.sh/anthropics/skills/pdf) | [github.com/anthropics/skills/tree/main/pdf](https://github.com/anthropics/skills/tree/main/pdf) |
| `xlsx` | Competitive matrices and library-evaluation tables ship as `.xlsx` when the human wants to filter/sort. | [skills.sh/anthropics/skills/xlsx](https://skills.sh/anthropics/skills/xlsx) | [github.com/anthropics/skills/tree/main/xlsx](https://github.com/anthropics/skills/tree/main/xlsx) |
| `skill-creator` | Required to author the four custom skills below — including the rubric. | [skills.sh/anthropics/skills/skill-creator](https://skills.sh/anthropics/skills/skill-creator) | [github.com/anthropics/skills/tree/main/skill-creator](https://github.com/anthropics/skills/tree/main/skill-creator) |
| `internal-comms` | Report tone & structure — status reports, FAQs, exec summaries. The "TL;DR ≤150 words" cadence in `instructions.md` lives here. | [skills.sh/anthropics/skills/internal-comms](https://skills.sh/anthropics/skills/internal-comms) | [github.com/anthropics/skills/tree/main/internal-comms](https://github.com/anthropics/skills/tree/main/internal-comms) |
| `docx` | Some reports (compliance-adjacent, customer-shareable) need to land as `.docx`, not markdown. | [skills.sh/anthropics/skills/docx](https://skills.sh/anthropics/skills/docx) | [github.com/anthropics/skills/tree/main/docx](https://github.com/anthropics/skills/tree/main/docx) |

> Mid-research notes go into a `NOTES.md` artifact in the agent's `work_dir`. The next Researcher task on the same `(agent, issue)` pair picks them up via Multica's session resumption (`multica.md` §5). No skill needed for that.

### To author via `skill-creator` (priority)

| Skill | Purpose |
|---|---|
| **`research-rubric`** ← **highest priority for this agent** | Encodes the 5-axis judge from `agent-orchestration.md` §4.7: **factual accuracy** · **citation accuracy** · **completeness** · **source quality** (primary > secondary > SEO content farm) · **tool efficiency**. Returns floats 0.0–1.0 per axis with one-sentence justifications. The agent self-scores every report; any axis < 0.6 triggers re-work of the weakest section. Critically: **the rubric must be a skill, not a prompt paragraph**, because (a) at Tier 3 split, both Market and Technical Researchers attach the same rubric — single source of truth, (b) the rubric is also the *judge prompt* used by Chief of Staff in the §6.4 multi-agent flow. |
| **`competitive-teardown-template`** | Repeatable structure for vendor matrices: feature columns, pricing column with currency + cadence, "moat" sentence, "why a customer leaves" sentence. |
| **`primary-source-rubric`** | Decision tree for "is this a primary source": vendor's own docs/blog/pricing/GitHub = primary · authoritative original research = primary · founder interviews on first-party podcasts = primary · listicle-SEO, AI-summary articles, Forbes/Inc contributor pieces = secondary at best. |
| **`research-brief-intake`** | Standard 4-question intake: (1) decision this enables (2) deadline (3) "no-go" sources for *this* brief (4) deliverable format. Removes the "what does the human actually want?" failure mode. |

## Tools & API access

**Hard rule for this agent**: read-heavy. The only writes are (a) Notion *Drafts* page (never *Published*), (b) the workspace `work_dir` filesystem. No GitHub write, no DB write, no Slack/email send.

### Recommendations

**Primary web search: Exa.** Genuinely free tier, no credit card; semantic search bias surfaces conceptual matches and primary sources better than keyword indexes. **Backup: Tavily.** 1,000 credits/month free, RAG-optimized.

**Why not Brave as primary**: Brave killed its free Search API tier in February 2026 — now $5/1k requests. For a solo founder running 5–10 research tasks/day, that's a real bill within a month. If you later want keyword-strict search to complement Exa's semantic bias, add Brave as a *third* option — don't make it primary.

**Why not Perplexity as primary**: Perplexity returns synthesized answers with citations. That's the *output* this agent produces — using it as a tool means delegating the agent's core judgment to a black box.

| Service | How the agent uses it | Required env vars | Scope |
|---|---|---|---|
| **Exa** | `curl https://api.exa.ai/search` and `/contents` for `web_search`, `research_paper_search`, `find_similar`. | `EXA_API_KEY` | Read-only. Free tier covers daily load. |
| **Tavily** | `curl https://api.tavily.com/search` and `/extract`. Backup search. | `TAVILY_API_KEY` (`tvly-…`) | Read-only. 1k credits/month free. |
| **Firecrawl** | `curl https://api.firecrawl.dev/v1/scrape` and `/crawl` for full-page scrape with JS rendering. Search snippets aren't enough for vendor pricing pages or competitive teardowns. | `FIRECRAWL_API_KEY` (`fc-…`) | Read-only. |
| **GitHub** | `gh` CLI for read-only repo metadata: `gh repo view`, `gh release list`, `gh issue list --state all`, `gh search code`. Library evaluations need: last-release date, open-issue ratio, CVE history, contributor count, commit cadence. | `GH_TOKEN=$GITHUB_PAT_RESEARCHER` (read-only: `metadata:read`, `contents:read`, `pull_requests:read`, `issues:read`) | Read-only. |
| **Notion** | Notion REST API: `curl https://api.notion.com/v1/pages` to create draft pages under the workspace's "Research Drafts" parent page. | `NOTION_TOKEN=secret_…` (integration token granted access to one parent page only) | Scoped write — drafts only, never "Published". |
| **Library docs** | Claude Code's WebFetch tool against vendor docs sites (Next.js, etc.) for technical evaluations. | none | n/a |
| **Hacker News** | Public HN API: `curl https://hn.algolia.com/api/v1/search?query=...` and `https://hacker-news.firebaseio.com/v0/`. | none (public) | Read-only. |
| **Reddit** | Reddit OAuth2 read-only: `curl https://oauth.reddit.com/r/<sub>/search.json` with `Authorization: Bearer <access-token>` (refresh via `/api/v1/access_token`). | `REDDIT_CLIENT_ID` · `REDDIT_CLIENT_SECRET` · `REDDIT_USER_AGENT` | Read-only. **No write tools.** |

### Wiring in Multica

In the Multica GUI (Settings → Agents → Researcher), set:

- **Custom arguments**: leave empty.
- **Environment** (one `KEY=VALUE` per line):
  ```
  EXA_API_KEY=...
  TAVILY_API_KEY=tvly-...
  FIRECRAWL_API_KEY=fc-...
  GH_TOKEN=ghp_...
  NOTION_TOKEN=secret_...
  REDDIT_CLIENT_ID=...
  REDDIT_CLIENT_SECRET=...
  REDDIT_USER_AGENT=multica-researcher/1.0 by <handle>
  ```

The agent invokes `gh`, `curl` from Bash. No MCP server config to maintain.

Rationale notes:
- **github**: PAT is read-only — `metadata:read`, `contents:read`, `pull_requests:read`, `issues:read`. Write paths are unreachable because the token cannot perform them.
- **notion**: Integration token is restricted at consent time to "Research Drafts" parent page; never grant access to "Published".
- **reddit**: client_credentials grant flow yields a read-only access token; the application should be registered as `script` type with no `submit`/`comment`/`message` scopes.

## Coherence check

The system prompt's three hard rules — *primary sources*, *no invented stats*, *what we don't know* — line up directly with three rubric axes (source quality, factual accuracy, completeness) encoded in the `research-rubric` skill. The tool set is read-heavy with one scoped write (Notion Drafts), matching the anti-scope clause that forbids autonomous publishing. Search is split across Exa (semantic, free) and Tavily (keyword backup, free) so the agent isn't single-vendor-locked or paying Brave's metered bills. Firecrawl and GitHub close the "search snippet wasn't enough" and "vendor blog post lied about activity" gaps. Reddit and HN unlock sentiment scans without scraping SEO summaries. At Tier 3 split, both Market and Technical Researchers inherit the same rubric — no rubric drift.

### Sources
- `agent-orchestration.md` §4.7 — 5-axis rubric definition.
- `agent-roster.md` §5.5 (role spec), §6.4 (multi-agent flow).
- `skills-bank.md` §1 (be-slightly-pushy rule), §3 (Tier-1 essentials).
- Exa API — https://docs.exa.ai/reference/getting-started
- Tavily API — https://docs.tavily.com/
- Firecrawl API — https://docs.firecrawl.dev/api-reference/introduction
- Notion API — https://developers.notion.com/reference/intro
- Reddit OAuth — https://www.reddit.com/dev/api/oauth
- Hacker News API — https://hn.algolia.com/api · https://github.com/HackerNews/API
- GitHub CLI — https://cli.github.com/manual/
