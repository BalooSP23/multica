# Skills + tools for Frontend Builder

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
| `webapp-testing` | Anthropic-flavored "spin up dev server, run Playwright, screenshot, report" loop — fires on *new* UI work. Overlaps `playwright-best-practices` ~30%; trigger verbs differ. | [skills.sh/anthropics/skills/webapp-testing](https://skills.sh/anthropics/skills/webapp-testing) | [github.com/anthropics/skills/tree/main/webapp-testing](https://github.com/anthropics/skills/tree/main/webapp-testing) |
| `web-artifacts-builder` | Quick prototypes / spike artifacts before committing to a real route. Optional but cheap (small L1). | [skills.sh/anthropics/skills/web-artifacts-builder](https://skills.sh/anthropics/skills/web-artifacts-builder) | [github.com/anthropics/skills/tree/main/web-artifacts-builder](https://github.com/anthropics/skills/tree/main/web-artifacts-builder) |

**Skills bundled with Claude Code** (auto-load when runtime is `claude-code`): `simplify`, `review`. Run on every PR.

**Skills explicitly NOT attached** (anti-scope hygiene): no `claude-api`, `supabase:*`, `stripe/*`, `vercel-functions`, `vercel-storage`, `vercel:auth`, `env-vars`, `vercel:deploy*`, `vercel:status`, `security-review` — those belong to Backend Builder / DevOps / Reviewer. Attaching them here floods L1 metadata and erodes trigger accuracy.

## Tools & API access

| Service | How the agent uses it | Required env vars | Scope |
|---|---|---|---|
| **GitHub** | `gh` CLI for branches, commits, `gh pr create`, `gh issue comment`. Never `gh pr merge`. | `GH_TOKEN=$GITHUB_PAT_FRONTEND_BUILDER` (`contents:write`, `pull_requests:write` no-merge, `issues:read`) | No merge, no force-push. Branch protection on `main` is the last line of defense. |
| **Playwright** | `npx playwright install` (once per task `work_dir`), `npx playwright test`, `npx playwright codegen` for new specs. Local headless Chromium. | none (uses workspace's `package.json`) | Task-scoped browser. |
| **shadcn** | `pnpm dlx shadcn@latest add <component>` (or `npx shadcn`) per the workspace's `components.json`. Custom registries: same CLI, registry URL in config. | none (registry URL lives in `components.json`) | Read public + workspace registries; writes only into the task `work_dir`. |
| **Vercel** | `vercel` CLI for `vercel ls` (list deployments), `vercel logs <url>` (build/runtime), `vercel inspect <url>`. Read-only deployment metadata. | `VERCEL_TOKEN` (read-only token via `vercel tokens create --scope <team>`) | Read-only. |
| **Library docs** | Claude Code's WebFetch tool against the official Next.js / React / Tailwind / shadcn docs sites. | none | n/a |
| **Filesystem** | Per-task `work_dir`. Read tokens from `app/globals.css`, write components, write specs. | `MULTICA_TASK_WORKDIR` (set by daemon) | Task-scoped only. |
| **Figma** | `curl https://api.figma.com/v1/files/<file-key>` with `X-Figma-Token: $FIGMA_PAT` to read file structure + Dev Mode tokens. PAT scoped to one team, read-only. | `FIGMA_PAT` (read-only, single-team) | Read-only Dev Mode equivalent. |

### Wiring in Multica

In the Multica GUI (Settings → Agents → Frontend Builder), set:

- **Custom arguments**: leave empty.
- **Environment** (one `KEY=VALUE` per line):
  ```
  GH_TOKEN=ghp_...
  VERCEL_TOKEN=...
  FIGMA_PAT=figd_...
  MULTICA_TASK_WORKDIR=/home/<user>/multica_workspaces/<task-id>
  ```

The daemon spawns the agent CLI with these env vars; the agent invokes `gh`, `npx`, `vercel`, `pnpm`, `curl` from Bash. No MCP server config to maintain.

Rationale:
- **GitHub** PAT scoped to `contents:write`, `pull_requests:write` (no merge), `issues:read`. Branch protection on `main` blocks merges from this PAT.
- **Vercel** token scope is the smallest one that allows reading deployments; never grant project-write or deploy.
- **Figma** PAT is single-team, read-only.

### Designer → Frontend handoff (how design tokens / Figma URLs arrive)

Three channels, in order of preference:

1. **Pinned `design/tokens.json` in the repo** (canonical). Designer agent commits a JSON token file at `design/tokens.json`; a generator step writes the `@theme inline` block in `app/globals.css`. This survives the Designer agent not existing yet on day one — Frontend authors the token file itself and the future Designer absorbs the contract.
2. **Figma URL in the Multica issue body** under a `Design:` line. Frontend reads the file via the Figma REST API; tokens are extracted and reconciled against `design/tokens.json` (mismatches file an issue against Designer).
3. **Attached file on the Multica issue** (PNG/PDF mock) — fallback when neither of the above is available; Frontend implements visually and notes "no Figma source" in the PR.

## Coherence check

The skill set covers the full loop: aesthetic decision (`frontend-design`) → composition (`shadcn`, `tailwind-v4-shadcn`, `ui-ux-pro-max`, `refactoring-ui`) → framework correctness (`nextjs`, `react-best-practices`, `next-cache-components`) → a11y (`accessibility-a11y`) → tests (`webapp-testing`, `playwright-best-practices`) → self-review (`simplify`, `review`). The tool set covers: code I/O (`gh`, filesystem), component supply chain (`shadcn` CLI), runtime feedback (`playwright`, `vercel` read-only), design-source ingestion (Figma REST), and current docs (WebFetch). No tool grants production write access; merge and infra mutation stay outside this agent.
