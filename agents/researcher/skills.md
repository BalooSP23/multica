# Skills + MCPs for Researcher

## Skills (Multica → `agent_skill` table)

The Researcher is light on skills, heavy on MCPs — most of the value is "find primary sources and read them carefully," which is MCP work. The skill list focuses on document I/O and the custom rubrics authored via `skill-creator`.

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

## MCP servers (`agent.mcp_config`)

**Hard rule for this agent**: read-heavy. The only writes are (a) Notion *Drafts* page (never *Published*), (b) the workspace `work_dir` filesystem. No GitHub write, no code-edit MCPs, no DB-write, no Slack/email send.

### Recommendations

**Primary web search: Exa.** Genuinely free tier, no credit card; semantic search bias surfaces conceptual matches and primary sources better than keyword indexes. Available as a hosted remote MCP at `https://mcp.exa.ai/mcp`. **Backup: Tavily.** 1,000 credits/month free, RAG-optimized.

**Why not Brave as primary**: Brave killed its free Search API tier in February 2026 — now $5/1k requests. For a solo founder running 5–10 research tasks/day, that's a real bill within a month. If you later want keyword-strict search to complement Exa's semantic bias, add Brave as a *third* MCP — don't make it primary.

**Why not Perplexity as primary**: Perplexity returns synthesized answers with citations. That's the *output* this agent produces — using it as a tool means delegating the agent's core judgment to a black box. Use sparingly via WebFetch on `perplexity.ai`, not as a standing MCP.

| MCP | Scope | Why for this role |
|---|---|---|
| **Exa MCP** (`mcp.exa.ai/mcp` — official, hosted) | read-only — `web_search`, `research_paper_search`, `company_research`, `crawl`, `find_similar` | Primary search. Semantic bias = primary-source bias. Free tier covers daily load. |
| **Tavily MCP** (`tavily-ai/tavily-mcp`) | read-only — `tavily-search`, `tavily-extract`, `tavily-crawl`, `tavily-map` | Backup search; useful when Exa rate-limits or for keyword-precision queries. |
| **Firecrawl MCP** (`firecrawl-mcp`) | read-only — `scrape`, `crawl`, `map`, `search`, `extract` | Deep-scrape full pages with JS rendering — search snippets aren't enough for vendor pricing pages or competitive teardowns. |
| **GitHub MCP** | **read-only** — repos, issues, PRs, releases, search code. Disable all write tools. | Library evaluations need: last-release date, open-issue ratio, CVE history, contributor count, commit cadence. |
| **Notion MCP** (`mcp.notion.com/mcp` — official, hosted, OAuth) | scoped write — only the workspace's "Research Drafts" parent page; *never* the "Published" page | Where reports land. Human reviews drafts and moves them to Published. |
| **context7 MCP** | read-only | Library docs at the version the workspace actually uses — for technical evaluations. |
| **Hacker News MCP** (`erithwik/mcp-hn`) | read-only — top/new/ask/show stories, comments, search | Sentiment + qualitative signal on technical/dev-tool topics. Free, no auth. |
| **Reddit MCP** (`karanb192/reddit-mcp-buddy`) | read-only — `search_posts`, `get_subreddit`, `get_comments` | Sentiment scans of ICP subreddits. **No write tools** — the agent never posts, comments, or DMs. |

### How to wire this in Multica

The cleaned, spec-valid config lives at `agents/researcher/mcp.json`. Multica's agent-settings GUI does not expose an MCP page today — [PR #1221](https://github.com/multica-ai/multica/pull/1221) is open. The daemon reads `agent.mcp_config` from the DB and passes it to Claude Code via `--mcp-config <tempfile>` (PR #1168, shipped v0.2.6). Full procedure in `agents/MCP-WIRING.md` Part A.

In the Multica GUI (Settings → Agents → Researcher):
- **Custom arguments**: leave empty.
- **Environment** (one `KEY=VALUE` per line — referenced as `${VAR}` in `mcp.json`):
  ```
  EXA_API_KEY=...
  TAVILY_API_KEY=tvly-...
  FIRECRAWL_API_KEY=fc-...
  GITHUB_PAT_RESEARCHER=ghp_...
  REDDIT_CLIENT_ID=...
  REDDIT_CLIENT_SECRET=...
  ```
  `notion` is OAuth-based — handshake via `/mcp` panel on first agent run; restrict the OAuth grant at consent time to "Research Drafts" parent page (do not grant access to "Published"). `hackernews` (`mcp-hn`) requires no auth.

Rationale notes:
- **github**: PAT is read-only — `metadata:read`, `contents:read`, `pull_requests:read`, `issues:read`. Write tools unreachable because the token cannot perform them.
- **notion**: OAuth scope must be restricted to "Research Drafts" parent page; never grant the integration access to "Published".
- **reddit**: read-only OAuth2; the `reddit-mcp-buddy` package's submit/comment/dm tools are not exposed via the standard read-only auth flow. Confirm at agent-runtime via `/mcp` listing.

## Coherence check

The system prompt's three hard rules — *primary sources*, *no invented stats*, *what we don't know* — line up directly with three rubric axes (source quality, factual accuracy, completeness) encoded in the `research-rubric` skill. The MCP set is read-heavy with one scoped write (Notion Drafts), matching the anti-scope clause that forbids autonomous publishing. Search is split across Exa (semantic, free) and Tavily (keyword backup, free) so the agent isn't single-vendor-locked or paying Brave's metered bills. Firecrawl and GitHub close the "search snippet wasn't enough" and "vendor blog post lied about activity" gaps. Reddit and HN unlock sentiment scans without scraping SEO summaries of those forums. At Tier 3 split, both Market and Technical Researchers inherit the same rubric — no rubric drift.

### Sources
- `agent-orchestration.md` §4.7 — 5-axis rubric definition.
- `agent-roster.md` §5.5 (role spec), §6.4 (multi-agent flow).
- `skills-bank.md` §1 (be-slightly-pushy rule), §3 (Tier-1 essentials).
- Exa MCP — https://github.com/exa-labs/exa-mcp-server · https://mcp.exa.ai/mcp
- Tavily MCP — https://github.com/tavily-ai/tavily-mcp
- Firecrawl MCP — https://github.com/firecrawl/firecrawl-mcp-server
- Brave Search MCP (not primary) — https://github.com/brave/brave-search-mcp-server
- Notion MCP (official) — https://mcp.notion.com/mcp
- Hacker News MCP — https://github.com/erithwik/mcp-hn
- Reddit MCP — https://github.com/karanb192/reddit-mcp-buddy
- GitHub MCP — https://github.com/github/github-mcp-server
