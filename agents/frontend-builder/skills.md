# Skills + MCPs for Frontend Builder

## Skills (Multica → `agent_skill` table)

Every skill below is imported into Multica via `multica skill import --url <skills.sh URL>`.

### To attach (import from skills.sh)

| Skill | Why for this role | skills.sh | GitHub fallback |
|---|---|---|---|
| `frontend-design` | Aesthetic discipline, anti-AI-slop, picks a direction (brutalist/editorial/luxury/etc.) before any JSX. | [skills.sh/anthropics/skills/frontend-design](https://skills.sh/anthropics/skills/frontend-design) | [github.com/anthropics/skills/tree/main/frontend-design](https://github.com/anthropics/skills/tree/main/frontend-design) |
| `ui-ux-pro-max` | Broad UI catalog (50 styles, 21 palettes, font pairings, charts) — composes with `frontend-design`. | [skills.sh/nextlevelbuilder/ui-ux-pro-max-skill/ui-ux-pro-max](https://skills.sh/nextlevelbuilder/ui-ux-pro-max-skill/ui-ux-pro-max) | [github.com/nextlevelbuilder/ui-ux-pro-max-skill](https://github.com/nextlevelbuilder/ui-ux-pro-max-skill) |
| `refactoring-ui` | Visual hierarchy, typography, layout, depth rules (Wathan & Schoger principles). | [skills.sh/wondelai/skills/refactoring-ui](https://skills.sh/wondelai/skills/refactoring-ui) | [github.com/wondelai/skills](https://github.com/wondelai/skills) |
| `tailwind-v4-shadcn` | Tailwind v4 + shadcn `@theme inline` patterns; prevents 8 documented config errors. | [skills.sh/jezweb/claude-skills/tailwind-v4-shadcn](https://skills.sh/jezweb/claude-skills/tailwind-v4-shadcn) | [github.com/jezweb/claude-skills](https://github.com/jezweb/claude-skills) |
| `shadcn` | Component composition, CLI, theming. | [skills.sh/shadcn/ui/shadcn](https://skills.sh/shadcn/ui/shadcn) | [github.com/shadcn-ui/ui](https://github.com/shadcn-ui/ui) |
| `shadcn` (Vercel-flavored) | Vercel-labs flavored shadcn skill — pairs with `shadcn` above (descriptions trigger on different verbs). | [skills.sh/vercel/vercel-plugin/shadcn](https://skills.sh/vercel/vercel-plugin/shadcn) | [github.com/vercel/vercel-plugin](https://github.com/vercel/vercel-plugin) |
| `react-best-practices` (Vercel) | TSX reviewer — fires after editing multiple TSX components (component structure, hooks, a11y, perf, TS patterns). | [skills.sh/vercel/vercel-plugin/react-best-practices](https://skills.sh/vercel/vercel-plugin/react-best-practices) | [github.com/vercel-labs/agent-skills](https://github.com/vercel-labs/agent-skills) |
| `nextjs` (Vercel) | App Router, RSC, routing, data fetching. | [skills.sh/vercel/vercel-plugin/nextjs](https://skills.sh/vercel/vercel-plugin/nextjs) | [github.com/vercel/vercel-plugin](https://github.com/vercel/vercel-plugin) |
| `next-cache-components` | RSC caching boundaries — keeps Frontend out of cache footguns. | [skills.sh/vercel/vercel-plugin/next-cache-components](https://skills.sh/vercel/vercel-plugin/next-cache-components) | [github.com/vercel-labs/next-skills](https://github.com/vercel-labs/next-skills) |
| `routing-middleware` | When middleware affects rendering on a touched route. | [skills.sh/vercel/vercel-plugin/routing-middleware](https://skills.sh/vercel/vercel-plugin/routing-middleware) | [github.com/vercel/vercel-plugin](https://github.com/vercel/vercel-plugin) |
| `turbopack` | Build-time issues during dev. | [skills.sh/vercel/vercel-plugin/turbopack](https://skills.sh/vercel/vercel-plugin/turbopack) | [github.com/vercel/vercel-plugin](https://github.com/vercel/vercel-plugin) |
| `accessibility-a11y` | WCAG 2.2 AA — keyboard, contrast, ARIA. | [skills.sh/mindrally/skills/accessibility-a11y](https://skills.sh/mindrally/skills/accessibility-a11y) | [github.com/mindrally/skills](https://github.com/mindrally/skills) |
| `playwright-best-practices` | Cross-browser e2e, auto-wait, test runner. | [skills.sh/currents-dev/playwright-best-practices-skill/playwright-best-practices](https://skills.sh/currents-dev/playwright-best-practices-skill/playwright-best-practices) | [github.com/currents-dev/playwright-best-practices-skill](https://github.com/currents-dev/playwright-best-practices-skill) |
| `webapp-testing` | Anthropic-flavored "spin up dev server, run Playwright, screenshot, report" loop — fires on *new* UI work where dev server isn't running yet. Overlaps `playwright-best-practices` ~30%; trigger verbs differ ("test this UI" vs. "write a Playwright spec"). | [skills.sh/anthropics/skills/webapp-testing](https://skills.sh/anthropics/skills/webapp-testing) | [github.com/anthropics/skills/tree/main/webapp-testing](https://github.com/anthropics/skills/tree/main/webapp-testing) |
| `web-artifacts-builder` | Quick prototypes / spike artifacts before committing to a real route. Optional but cheap (small L1). | [skills.sh/anthropics/skills/web-artifacts-builder](https://skills.sh/anthropics/skills/web-artifacts-builder) | [github.com/anthropics/skills/tree/main/web-artifacts-builder](https://github.com/anthropics/skills/tree/main/web-artifacts-builder) |

**Skills bundled with Claude Code** (no marketplace import; auto-load when runtime is `claude-code`): `simplify`, `review`. Run on every PR.

**Skills explicitly NOT attached** (anti-scope hygiene): no `claude-api`, `supabase:*`, `stripe/*`, `vercel-functions`, `vercel-storage`, `vercel:auth`, `env-vars`, `vercel:deploy*`, `vercel:status`, `security-review` — those belong to Backend Builder / DevOps / Reviewer. Attaching them here floods L1 metadata and erodes trigger accuracy (anti-pattern §7 in `agent-roster.md`). The user's locally-authored `ui-ux-expert` skill is also not attached — it's a private/laptop-only skill; if you want it on Multica, publish it via `skill-creator` first.

## MCP servers (`agent.mcp_config`)

| MCP | Scope | Why |
|---|---|---|
| `github` | **PR + issue comment only** — no merge, no force-push, no branch delete. | Open PRs, push branches, comment on issues. Merge stays human. |
| `playwright` | local headless Chromium | Run e2e specs, take screenshots for visual regression diffs, capture a11y trees. |
| `shadcn` | read shadcn registries, run `shadcn add <component>` against the workspace | Install components by name from the issue ("add a login form"); supports private/custom registries via `components.json`. |
| `vercel` | read deployments, fetch preview URL, read build/runtime logs (no project mutation) | Pin preview URL on PR, debug failed previews. |
| `context7` | read-only docs lookup | Up-to-date Next.js / React / Tailwind v4 / shadcn docs newer than training cut-off. |
| `filesystem` | scoped to workspace `work_dir` (Multica per-task) | Read tokens from `app/globals.css`, write components, write specs. |
| `figma` | read-only — Dev Mode | Consume Figma URLs the Designer agent (Tier 2) drops in issue bodies. **Recommend the official Figma Dev Mode MCP** (`https://mcp.figma.com/mcp`) — supported, has variables/auto-layout/code-connect; the GLips fork is read-context-only and Cursor-tuned. |

### How to wire this in Multica

The cleaned, spec-valid config lives at `agents/frontend-builder/mcp.json`. Multica's agent-settings GUI does not expose an MCP page today — [PR #1221](https://github.com/multica-ai/multica/pull/1221) is open. The daemon reads `agent.mcp_config` from the DB and passes it to Claude Code via `--mcp-config <tempfile>` (PR #1168, shipped v0.2.6). Full procedure in `agents/MCP-WIRING.md` Part A.

In the Multica GUI (Settings → Agents → Frontend Builder):
- **Custom arguments**: leave empty, or use for non-MCP tuning.
- **Environment** (one `KEY=VALUE` per line — referenced as `${VAR}` in `mcp.json`):
  ```
  GITHUB_PAT_FRONTEND_BUILDER=ghp_...
  MULTICA_TASK_WORKDIR=/home/<user>/multica_workspaces/<task-id>
  ```
  `vercel` and `figma` are OAuth-based hosted MCPs (handshake via `/mcp` panel on first connection) — no env var needed. `playwright`, `shadcn`, `context7` need no auth.

Rationale notes (Claude Code's `mcp.json` schema does not accept `tool_filter` — tool restriction is enforced at PAT scope and Reviewer-agent gating instead):
- **github**: PAT scoped to `contents:write`, `pull_requests:write` (no merge), `issues:read`. Branch protection on `main` blocks merges from this PAT.
- **vercel**: read-only via OAuth scope at consent time — grant only deployment-read scopes; do not grant project-write or deploy.
- **figma**: read-only Dev Mode scope.

### Designer → Frontend handoff (how design tokens / Figma URLs arrive)

Three channels, in order of preference:

1. **Pinned `design/tokens.json` in the repo** (canonical). Designer agent commits a JSON token file at `design/tokens.json`; a generator step writes the `@theme inline` block in `app/globals.css`. This survives the Designer agent not existing yet on day one — Frontend authors the token file itself and the future Designer absorbs the contract.
2. **Figma URL in the Multica issue body** under a `Design:` line. Frontend's Figma MCP reads the file via Dev Mode; tokens are extracted and reconciled against `design/tokens.json` (mismatches file an issue against Designer).
3. **Attached file on the Multica issue** (PNG/PDF mock) — fallback when neither of the above is available; Frontend implements visually and notes "no Figma source" in the PR.

## Coherence check

The skill set covers the full loop: aesthetic decision (`frontend-design`) → composition (`shadcn`, `tailwind-v4-shadcn`, `ui-ux-pro-max`, `refactoring-ui`) → framework correctness (`nextjs`, `react-best-practices`, `next-cache-components`) → a11y (`accessibility-a11y`) → tests (`webapp-testing` for first-time UI, `playwright-best-practices` for spec-authoring) → self-review (`simplify`, `review`). MCP set covers: code I/O (`github`, `filesystem`), component supply chain (`shadcn`), runtime feedback (`playwright`, `vercel`), design-source ingestion (`figma`), and current docs (`context7`). No MCP grants production write access; merge and infra mutation stay outside this agent.
