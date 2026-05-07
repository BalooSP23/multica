# Skills + MCPs for Researcher

## Skills (Multica → `agent_skill` table)

### Already installed locally (just attach)
The Researcher is light on skills, heavy on MCPs — most of the value is "find primary sources and read them carefully," which is MCP work, not skill work. Of the user's locally-installed Anthropic-ecosystem skills, only `pdf` and `xlsx` are both portable into Multica's `agent_skill` table *and* genuinely useful for this role; the rest of the lift comes from the four custom skills below built via `skill-creator`.

| Skill | Source | skills.sh | clawhub.io | Why for this role |
|---|---|---|---|---|
| `pdf` | `anthropics/skills/pdf` | [skills.sh/anthropics/skills/pdf](https://skills.sh/anthropics/skills/pdf) | unreachable (cert/domain expired May 2026) — fallback: [github.com/anthropics/skills/tree/main/pdf](https://github.com/anthropics/skills/tree/main/pdf) | Primary sources are often PDFs (whitepapers, S-1s, vendor security docs, academic research). Without this, the agent skips PDF citations or hallucinates summaries of them. `skills-bank.md` §3. |
| `xlsx` | `anthropics/skills/xlsx` | [skills.sh/anthropics/skills/xlsx](https://skills.sh/anthropics/skills/xlsx) | unreachable — fallback: [github.com/anthropics/skills/tree/main/xlsx](https://github.com/anthropics/skills/tree/main/xlsx) | Competitive matrices and library-evaluation tables are easier to ship as `.xlsx` when the human wants to filter/sort. `skills-bank.md` §3. |

> **Note**: an earlier draft listed `gsd-note` here for mid-research observation capture. That's a Claude Code harness skill bound to the user's local terminal session — Multica doesn't import it into agent task `work_dir`s. The replacement is simpler: the agent appends mid-research observations to a `NOTES.md` artifact in its `work_dir`, which the next Researcher task on the same `(agent, issue)` pair picks up via Multica's session resumption (`multica.md` §5). No skill needed.

### Needs installation
| Skill | Source | skills.sh | clawhub.io | Why for this role |
|---|---|---|---|---|
| `skill-creator` | `anthropics/skills/skill-creator` | [skills.sh/anthropics/skills/skill-creator](https://skills.sh/anthropics/skills/skill-creator) | unreachable (cert/domain expired May 2026) — fallback: [github.com/anthropics/skills/tree/main/skill-creator](https://github.com/anthropics/skills/tree/main/skill-creator) | Required to author the four custom skills below — including the rubric. `skills-bank.md` §3, Tier 1 essential. |
| `internal-comms` | `anthropics/skills/internal-comms` | [skills.sh/anthropics/skills/internal-comms](https://skills.sh/anthropics/skills/internal-comms) | unreachable — fallback: [github.com/anthropics/skills/tree/main/internal-comms](https://github.com/anthropics/skills/tree/main/internal-comms) | Report tone & structure — status reports, FAQs, exec summaries. The "TL;DR ≤150 words" cadence in `instructions.md` lives here. `skills-bank.md` §3 / Tier 3. |
| `docx` | `anthropics/skills/docx` | [skills.sh/anthropics/skills/docx](https://skills.sh/anthropics/skills/docx) | unreachable — fallback: [github.com/anthropics/skills/tree/main/docx](https://github.com/anthropics/skills/tree/main/docx) | Some reports (compliance-adjacent, customer-shareable) need to land as `.docx`, not markdown. `skills-bank.md` §3. |

### To author via `skill-creator` (priority)
| Skill (proposed) | Purpose |
|---|---|
| **`research-rubric`** ← **highest priority for this agent** | Encodes the 5-axis judge from `agent-orchestration.md` §4.7: **factual accuracy** (claims match cited sources) · **citation accuracy** (sources actually say what's claimed) · **completeness** (covers the brief) · **source quality** (primary > secondary > SEO content farm) · **tool efficiency** (minimal redundant searches). The skill returns floats 0.0–1.0 per axis with a one-sentence justification. The agent self-scores every report; any axis < 0.6 triggers re-work of the weakest section. Critically: **the rubric must be a skill, not a system-prompt paragraph**, because (a) at Tier 3 split, both Market and Technical Researchers attach the same rubric — single source of truth, (b) the rubric is also the *judge prompt* used by Chief of Staff in the §6.4 multi-agent flow, and (c) skill descriptions trigger more reliably than mid-prompt instructions per `skills-bank.md` §1's "be slightly pushy" rule. |
| **`competitive-teardown-template`** | Repeatable structure for vendor matrices: feature column inventory, pricing column with currency + cadence, "moat" sentence, "why a customer leaves" sentence. Stops every teardown from being shaped differently. |
| **`primary-source-rubric`** | Decision tree for "is this a primary source": vendor's own docs/blog/pricing/GitHub = primary · authoritative original research = primary · founder interviews on first-party podcasts = primary · listicle-SEO, AI-summary articles, Forbes/Inc contributor pieces = secondary at best. Encodes the "no-go sources" workspace list. |
| **`research-brief-intake`** | Standard 4-question intake the agent runs before any research issue: (1) decision this enables (2) deadline (3) "no-go" sources for *this* brief on top of workspace defaults (4) deliverable format. Removes the "what does the human actually want?" failure mode from `agent-orchestration.md` §5. |

## MCP servers (`agent.mcp_config`)

**Hard rule for this agent**: read-heavy. The only writes are (a) Notion *Drafts* page (never *Published*), (b) the workspace `work_dir` filesystem. No GitHub write, no code-edit MCPs, no DB-write, no Slack/email send.

### Recommendations

**Primary web search: Exa.** Recommended because (a) genuinely free tier with no credit card (a key ICP fit for a solo-founder workspace), (b) semantic search bias surfaces conceptual matches and primary sources better than keyword indexes — which aligns with the "primary > SEO" house rule, (c) available as a hosted remote MCP at `https://mcp.exa.ai/mcp` so no local Node process to babysit. **Backup: Tavily.** 1,000 credits/month free, RAG-optimized, native LangChain/LlamaIndex integration if the workspace later builds an in-product research feature.

**Why not Brave as primary**: Brave killed its free Search API tier in February 2026 — it's now $5/1k requests with a $5 monthly credit and required public attribution to keep the credit. For a solo founder running 5–10 research tasks/day, that's a real bill within a month. Exa's free tier covers the same workloads at zero cost. If the workspace later wants keyword-strict search to complement Exa's semantic bias, add Brave as a *third* MCP — don't make it primary.

**Why not Perplexity as primary**: Perplexity returns synthesized answers with citations. That's the *output* this agent produces — using it as a tool means delegating the agent's core judgment to a black box. Use it sparingly (via WebFetch on `perplexity.ai` if needed), not as a standing MCP.

| MCP | Scope | Why for this role |
|---|---|---|
| **Exa MCP** (`mcp.exa.ai/mcp` — official, hosted) | read-only — `web_search`, `research_paper_search`, `company_research`, `crawl`, `find_similar` | Primary search. Semantic bias = primary-source bias. Free tier covers the daily research load. |
| **Tavily MCP** (`tavily-ai/tavily-mcp`) | read-only — `tavily-search`, `tavily-extract`, `tavily-crawl`, `tavily-map` | Backup search; useful when Exa rate-limits or for queries where keyword precision matters. 1k credits/month free. |
| **Firecrawl MCP** (`firecrawl/firecrawl-mcp-server`, npm `firecrawl-mcp`) | read-only — `scrape`, `crawl`, `map`, `search`, `extract` | Deep-scrape full pages with JS rendering — search snippets aren't enough for vendor pricing pages, changelog mining, or competitive teardowns where the actual claim is below the fold. |
| **GitHub MCP** (`github/github-mcp-server`) | **read-only** — repos, issues, PRs, releases, search code. Disable all write tools. | Library evaluations need: last-release date, open-issue ratio, CVE history, contributor count, commit cadence. None of this is reliable from search snippets. |
| **Notion MCP** (`mcp.notion.com/mcp` — official, hosted, OAuth) | scoped write — only the workspace's "Research Drafts" parent page; *never* the "Published" page | Where reports land. Human reviews drafts and moves them to Published. Confirmed available; user's tool list shows `mcp__claude_ai_Notion__*` is wired up. |
| **context7 MCP** (`context7`) | read-only | Library docs at the version the workspace actually uses — for technical evaluations. Already familiar to other Tier-1 agents in the roster, no new auth surface. |
| **Hacker News MCP** (community: `erithwik/mcp-hn`) | read-only — top/new/ask/show stories, comments, search | Sentiment + qualitative signal on technical/dev-tool topics. Free, no auth (public HN API). |
| **Reddit MCP** (community, OAuth2 read-only — e.g. `karanb192/reddit-mcp-buddy` or `enisze/reddit-mcp`) | read-only — `search_posts`, `get_subreddit`, `get_comments` | Sentiment scans of ICP subreddits. **No write tools** — the agent never posts, comments, or DMs. Treat as part of the trusted compute base per `skills-bank.md` §11. |

### How to wire this in Multica (no MCP UI page yet)

The cleaned, spec-valid config lives at `agents/researcher/mcp.json` (sibling file). Multica's agent-settings GUI does not expose an MCP page today — [PR #1221](https://github.com/multica-ai/multica/pull/1221) is still open as of May 2026. The daemon DOES however read `agent.mcp_config` from the DB and pass it to Claude Code via `--mcp-config <tempfile>` (PR #1168, shipped v0.2.6). So:

1. **Load `mcp.json` into the agent's `mcp_config` column** — via REST API, `multica` CLI, or direct SQL. Recipes in `agents/MCP-WIRING.md` Part A2. Custom arguments cannot inject `--mcp-config` (the daemon blocks it via `claudeBlockedArgs`).

2. **In the Multica GUI** (Settings → Agents → Researcher):
   - **Custom arguments**: leave empty.
   - **Environment** (one `KEY=VALUE` per line — secrets used as `${VAR}` in `mcp.json`):
     ```
     EXA_API_KEY=...
     TAVILY_API_KEY=tvly-...
     FIRECRAWL_API_KEY=fc-...
     GITHUB_PAT_RESEARCHER=ghp_...
     REDDIT_CLIENT_ID=...
     REDDIT_CLIENT_SECRET=...
     ```
     `notion` is OAuth-based — handshake completes once via `/mcp` panel on first agent run; restrict the OAuth grant at consent time to the workspace's "Research Drafts" parent page (do not grant access to "Published"). `hackernews` (`mcp-hn`) requires no auth (public HN API).

Rationale notes that previously lived in `_allowed_tools` / `_scope_note` pseudo-fields:
- **github**: PAT is read-only — `metadata:read`, `contents:read`, `pull_requests:read`, `issues:read`. Write tools are unreachable because the token cannot perform them.
- **notion**: OAuth scope must be restricted to "Research Drafts" parent page; never grant the integration access to "Published".
- **reddit**: read-only OAuth2; the `reddit-mcp-buddy` package's submit/comment/dm tools are not exposed via the standard read-only auth flow. Confirm at agent-runtime via `/mcp` listing.

## Coherence check
The system prompt's three hard rules — *primary sources*, *no invented stats*, *what we don't know* — line up directly with three rubric axes (source quality, factual accuracy, completeness) encoded in the `research-rubric` skill. The MCP set is read-heavy with one scoped write (Notion Drafts), matching the anti-scope clause that forbids autonomous publishing. Search is split across Exa (semantic, free) and Tavily (keyword backup, free) so the agent isn't single-vendor-locked or paying Brave's metered bills. Firecrawl and GitHub close the "search snippet wasn't enough" and "vendor blog post lied about activity" gaps that kill research-report credibility. Reddit and HN MCPs unlock sentiment scans without forcing the agent to scrape SEO summaries of those forums. Leverage compounds at Tier 3 split: both Market and Technical Researchers inherit the same rubric skill — no rubric drift — and just narrow their MCP sets (Market drops context7/GitHub-deep, Technical drops Reddit-sentiment).

### Sources
- `agent-orchestration.md` §4.7 — 5-axis rubric definition.
- `agent-roster.md` §5.5 (role spec), §6.4 (multi-agent flow), §7 (anti-pattern: chasing +90.2% on routine).
- `skills-bank.md` §1 (be-slightly-pushy rule), §3 (Tier-1 essentials), §6 (Firecrawl, Brave, Composio), §11 (security checklist).
- Exa MCP — https://github.com/exa-labs/exa-mcp-server · https://mcp.exa.ai/mcp (free tier, no key required for basic tools).
- Tavily MCP — https://github.com/tavily-ai/tavily-mcp · 1,000 credits/month free.
- Firecrawl MCP — https://github.com/firecrawl/firecrawl-mcp-server · `npx -y firecrawl-mcp`.
- Brave Search MCP (not primary) — https://github.com/brave/brave-search-mcp-server · *paid since Feb 2026* (https://www.implicator.ai/brave-drops-free-search-api-tier-puts-all-developers-on-metered-billing/).
- Notion MCP (official) — https://mcp.notion.com/mcp · https://developers.notion.com/guides/mcp/get-started-with-mcp · OAuth-scoped.
- Hacker News MCP — https://github.com/erithwik/mcp-hn (public API, no auth).
- Reddit MCP — https://github.com/karanb192/reddit-mcp-buddy · OAuth2 read-only.
- GitHub MCP — https://github.com/github/github-mcp-server.
