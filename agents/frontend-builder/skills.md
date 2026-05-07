# Skills + MCPs for Frontend Builder

## Skills

### Already installed locally (just attach)

> Note on URL columns: "local" means the skill is on the user's laptop already. Multica still imports each one into its own `skill` table — `simplify`, `review` are bundled with Claude Code itself and need no marketplace import. `ui-ux-expert` is a custom user-authored skill and has no public URL.

| Skill | Source | skills.sh | clawhub.io | Why |
|---|---|---|---|---|
| `frontend-design:frontend-design` | `anthropics/skills` | [skills.sh/anthropics/skills/frontend-design](https://skills.sh/anthropics/skills/frontend-design) | unreachable (cert/domain expired May 2026) — fallback: [github.com/anthropics/skills/tree/main/frontend-design](https://github.com/anthropics/skills/tree/main/frontend-design) | Aesthetic discipline, anti-AI-slop, picks a direction (brutalist/editorial/luxury/etc.) before any JSX |
| `ui-ux-pro-max:ui-ux-pro-max` | `nextlevelbuilder/ui-ux-pro-max-skill` | [skills.sh/nextlevelbuilder/ui-ux-pro-max-skill/ui-ux-pro-max](https://skills.sh/nextlevelbuilder/ui-ux-pro-max-skill/ui-ux-pro-max) | unreachable — fallback: [github.com/nextlevelbuilder/ui-ux-pro-max-skill](https://github.com/nextlevelbuilder/ui-ux-pro-max-skill) | Broad UI catalog (50 styles, 21 palettes, font pairings, charts) — composes with `frontend-design` |
| `refactoring-ui` | `wondelai/skills` (community port of Wathan & Schoger principles) | [skills.sh/wondelai/skills/refactoring-ui](https://skills.sh/wondelai/skills/refactoring-ui) | unreachable — fallback: [github.com/wondelai/skills](https://github.com/wondelai/skills) | Visual hierarchy, typography, layout, depth rules |
| `tailwind-v4-shadcn` | `jezweb/claude-skills` | [skills.sh/jezweb/claude-skills/tailwind-v4-shadcn](https://skills.sh/jezweb/claude-skills/tailwind-v4-shadcn) | unreachable — fallback: [github.com/jezweb/claude-skills](https://github.com/jezweb/claude-skills) | Tailwind v4 + shadcn `@theme inline` patterns; prevents 8 documented config errors |
| `shadcn` (shadcn/ui) | `shadcn/ui` | [skills.sh/shadcn/ui/shadcn](https://skills.sh/shadcn/ui/shadcn) | unreachable — fallback: [github.com/shadcn-ui/ui](https://github.com/shadcn-ui/ui) | Component composition, CLI, theming |
| `vercel:shadcn` | `vercel/vercel-plugin` | [skills.sh/vercel/vercel-plugin/shadcn](https://skills.sh/vercel/vercel-plugin/shadcn) | unreachable — fallback: [github.com/vercel/vercel-plugin](https://github.com/vercel/vercel-plugin) | Vercel-labs flavored shadcn skill — pairs with `shadcn` above (the user has both; descriptions trigger on different verbs). |
| `vercel:react-best-practices` | `vercel/vercel-plugin` (also republished as `vercel-react-best-practices` in `vercel-labs/agent-skills`) | [skills.sh/vercel/vercel-plugin/react-best-practices](https://skills.sh/vercel/vercel-plugin/react-best-practices) (alt: [skills.sh/vercel-labs/agent-skills/vercel-react-best-practices](https://skills.sh/vercel-labs/agent-skills/vercel-react-best-practices)) | unreachable — fallback: [github.com/vercel-labs/agent-skills](https://github.com/vercel-labs/agent-skills) | TSX reviewer — fires after editing multiple TSX components (component structure, hooks, a11y, perf, TS patterns) |
| `vercel:nextjs` | `vercel/vercel-plugin` | [skills.sh/vercel/vercel-plugin/nextjs](https://skills.sh/vercel/vercel-plugin/nextjs) | unreachable — fallback: [github.com/vercel/vercel-plugin](https://github.com/vercel/vercel-plugin) | App Router, RSC, routing, data fetching |
| `vercel:next-cache-components` | `vercel/vercel-plugin` (also `vercel-labs/next-skills`) | [skills.sh/vercel/vercel-plugin/next-cache-components](https://skills.sh/vercel/vercel-plugin/next-cache-components) (alt: [skills.sh/vercel-labs/next-skills/next-cache-components](https://skills.sh/vercel-labs/next-skills/next-cache-components)) | unreachable — fallback: [github.com/vercel-labs/next-skills](https://github.com/vercel-labs/next-skills) | RSC caching boundaries — keeps Frontend out of the cache footguns |
| `vercel:routing-middleware` | `vercel/vercel-plugin` | [skills.sh/vercel/vercel-plugin/routing-middleware](https://skills.sh/vercel/vercel-plugin/routing-middleware) | unreachable — fallback: [github.com/vercel/vercel-plugin](https://github.com/vercel/vercel-plugin) | When middleware affects rendering on a touched route |
| `vercel:turbopack` | `vercel/vercel-plugin` | [skills.sh/vercel/vercel-plugin/turbopack](https://skills.sh/vercel/vercel-plugin/turbopack) | unreachable — fallback: [github.com/vercel/vercel-plugin](https://github.com/vercel/vercel-plugin) | Build-time issues during dev |
| `accessibility-a11y` | `mindrally/skills` | [skills.sh/mindrally/skills/accessibility-a11y](https://skills.sh/mindrally/skills/accessibility-a11y) | unreachable — fallback: [github.com/mindrally/skills](https://github.com/mindrally/skills) | WCAG 2.2 AA — keyboard, contrast, ARIA |
| `playwright-e2e-testing` | `currents-dev/playwright-best-practices-skill` | [skills.sh/currents-dev/playwright-best-practices-skill/playwright-best-practices](https://skills.sh/currents-dev/playwright-best-practices-skill/playwright-best-practices) | unreachable — fallback: [github.com/currents-dev/playwright-best-practices-skill](https://github.com/currents-dev/playwright-best-practices-skill) | Cross-browser e2e, auto-wait, test runner |
| `simplify` | bundled with Claude Code | n/a (bundled — no marketplace import) | n/a (bundled) | Reuse / quality / efficiency review pre-commit |
| `review` | bundled with Claude Code | n/a (bundled) | n/a (bundled) | Self-review before pushing PR (final pass) |
| `ui-ux-expert` | user-authored locally | not on skills.sh (private/local) | unreachable — no public URL; if you want it shareable, publish via `skill-creator` first | Secondary UI sanity check (overlap with `ui-ux-pro-max` is acceptable; descriptions trigger differently) |

### Needs installation

| Skill | Source | skills.sh | clawhub.io | Why |
|---|---|---|---|---|
| `anthropics/webapp-testing` | `anthropics/skills/webapp-testing` | [skills.sh/anthropics/skills/webapp-testing](https://skills.sh/anthropics/skills/webapp-testing) | unreachable (cert/domain expired May 2026) — fallback: [github.com/anthropics/skills/tree/main/webapp-testing](https://github.com/anthropics/skills/tree/main/webapp-testing) | **Decision: install both `webapp-testing` and `playwright-e2e-testing`, but trigger them differently.** `webapp-testing` is the Anthropic-flavoured "spin up dev server, run Playwright agent against it, screenshot, report" loop — fires on *new* UI work where the dev server isn't running yet. `playwright-e2e-testing` is the broader cross-browser spec-authoring skill — fires when adding/modifying spec files in `tests/e2e/`. They overlap ~30%; that's tolerable because their L1 descriptions trigger on different verbs ("test this UI" vs. "write a Playwright spec"). |
| `anthropics/web-artifacts-builder` | `anthropics/skills/web-artifacts-builder` | [skills.sh/anthropics/skills/web-artifacts-builder](https://skills.sh/anthropics/skills/web-artifacts-builder) | unreachable — fallback: [github.com/anthropics/skills/tree/main/web-artifacts-builder](https://github.com/anthropics/skills/tree/main/web-artifacts-builder) | Quick prototypes / spike artifacts before committing to a real route. Optional but cheap (small L1). |

**Skills explicitly NOT attached** (anti-scope hygiene):
- No `claude-api`, `supabase:*`, `stripe/*`, `vercel:vercel-functions`, `vercel:vercel-storage`, `vercel:auth`, `vercel:env*`, `vercel:deploy*`, `vercel:status`, `vercel:verification`, `security-review` — those belong to Backend Builder / DevOps / Reviewer. Attaching them here floods L1 metadata and erodes trigger accuracy (anti-pattern §7 in `agent-roster.md`).

## MCP servers (`agent.mcp_config`)

| MCP | Scope | Why |
|---|---|---|
| `github` | **PR + issue comment only** — no merge, no force-push, no branch delete | Open PRs, push branches, comment on issues. Merge stays human. |
| `playwright` | local headless Chromium | Run e2e specs, take screenshots for visual regression diffs, capture a11y trees |
| `shadcn` | read shadcn registries, run `shadcn add <component>` against the workspace | Install components by name from the issue ("add a login form"); supports private/custom registries via `components.json` ([docs](https://ui.shadcn.com/docs/mcp)) |
| `vercel` | read deployments, fetch preview URL, read build/runtime logs (no project mutation) | Pin preview URL on PR, debug failed previews. Use the user's already-authorized `mcp__claude_ai_Vercel__*` tool surface — `get_deployment`, `get_deployment_build_logs`, `list_deployments`, `get_runtime_logs`, `web_fetch_vercel_url`, `search_vercel_documentation` ([docs](https://vercel.com/docs/mcp/vercel-mcp)) |
| `context7` | read-only docs lookup | Up-to-date Next.js / React / Tailwind v4 / shadcn docs that may be newer than training cut-off |
| `filesystem` | scoped to workspace `work_dir` (Multica per-task) | Read tokens from `app/globals.css`, write components, write specs |
| `figma` | read-only — Dev Mode | Consume Figma URLs the Designer agent (Tier 2) drops in issue bodies. **Recommend the official Figma Dev Mode MCP** (`https://mcp.figma.com/mcp` remote, or local `figma-developer-mcp`) over `glips/figma-context-mcp` — official is supported, has variables/auto-layout/code-connect, and the GLips fork is read-context-only and Cursor-tuned ([Figma docs](https://help.figma.com/hc/en-us/articles/32132100833559-Guide-to-the-Figma-MCP-server), [GLips repo](https://github.com/GLips/Figma-Context-MCP)). |

### How to wire this in Multica (no MCP UI page yet)

The cleaned, spec-valid config lives at `agents/frontend-builder/mcp.json` (sibling file). Multica's agent-settings GUI does not expose an MCP page today — [PR #1221](https://github.com/multica-ai/multica/pull/1221) is still open as of May 2026. The daemon DOES however read `agent.mcp_config` from the DB and pass it to Claude Code via `--mcp-config <tempfile>` (PR #1168, shipped v0.2.6). So:

1. **Load `mcp.json` into the agent's `mcp_config` column** — via REST API, `multica` CLI, or direct SQL. Recipes in `agents/MCP-WIRING.md` Part A2. Custom arguments cannot inject `--mcp-config` (the daemon blocks it via `claudeBlockedArgs`).

2. **In the Multica GUI** (Settings → Agents → Frontend Builder):
   - **Custom arguments**: leave empty, or use for non-MCP tuning.
   - **Environment** (one `KEY=VALUE` per line — secrets used as `${VAR}` in `mcp.json`):
     ```
     GITHUB_PAT_FRONTEND_BUILDER=ghp_...
     MULTICA_TASK_WORKDIR=/home/<user>/multica_workspaces/<task-id>
     ```
     `vercel` and `figma` are OAuth-based hosted MCPs (handshake completes via `/mcp` panel on first connection) — no env var needed for them. `playwright`, `shadcn`, `context7` need no auth.

Rationale notes that previously lived in `tool_filter` pseudo-fields (Claude Code's `mcp.json` schema does not accept `tool_filter` — tool restriction is enforced at PAT scope and Reviewer-agent gating instead):
- **github**: PAT scoped to `contents:write`, `pull_requests:write` (no merge), `issues:read`. Branch protection on `main` blocks merges from this PAT.
- **vercel**: read-only by virtue of OAuth scope at consent time — grant only deployment-read scopes; do not grant project-write or deploy.
- **figma**: read-only Dev Mode scope.

### Designer → Frontend handoff (how design tokens / Figma URLs arrive)

Three channels, in order of preference:

1. **Pinned `design/tokens.json` in the repo** (canonical). Designer agent commits a JSON token file at `design/tokens.json`; a generator step writes the `@theme inline` block in `app/globals.css`. This survives the Designer agent not existing yet on day one — Frontend authors the token file itself and the future Designer absorbs the contract.
2. **Figma URL in the Multica issue body** under a `Design:` line. Frontend's Figma MCP reads the file via Dev Mode; tokens are extracted and reconciled against `design/tokens.json` (mismatches file an issue against Designer).
3. **Attached file on the Multica issue** (PNG/PDF mock) — fallback when neither of the above is available; Frontend implements visually and notes "no Figma source" in the PR.

## Coherence check

The skill set covers the full loop: aesthetic decision (`frontend-design`) → composition (`shadcn`, `tailwind-v4-shadcn`, `ui-ux-pro-max`, `refactoring-ui`) → framework correctness (`vercel:nextjs`, `vercel:react-best-practices`, `vercel:next-cache-components`) → a11y (`accessibility-a11y`) → tests (`webapp-testing` for first-time UI, `playwright-e2e-testing` for spec-authoring) → self-review (`simplify`, `review`). MCP set covers: code I/O (`github`, `filesystem`), component supply chain (`shadcn`), runtime feedback (`playwright`, `vercel` deployments), design-source ingestion (`figma`), and current docs (`context7`). No MCP grants production write access; merge and infra mutation stay outside this agent.
