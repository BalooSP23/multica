# Skills + MCPs for Reviewer

## Skills (Multica → `agent_skill` table)

Reviewer runs on Codex CLI, but Multica still imports skills via `multica skill import --url <URL>` — the daemon writes them to `$CODEX_HOME/skills/<name>/` (`multica.md` §7). For a Codex-runtime agent, install community equivalents of `review`/`security-review`/`simplify` since they aren't part of Codex's bundle.

### To attach (import from skills.sh)

| Skill | Why for this role | skills.sh | GitHub fallback |
|---|---|---|---|
| `code-review-excellence` | Default code-review skill — reads diff, comments by line, summarizes verdict. Codex equivalent of Claude Code's bundled `review`. | [skills.sh/wshobson/agents/code-review-excellence](https://skills.sh/wshobson/agents/code-review-excellence) | [github.com/wshobson/agents](https://github.com/wshobson/agents) |
| `code-review-expert` | Second review-skill option — install one of `code-review-excellence` or `code-review-expert`, not both (they fight for the same trigger). | [skills.sh/sanyuan0704/code-review-expert/code-review-expert](https://skills.sh/sanyuan0704/code-review-expert/code-review-expert) | [github.com/sanyuan0704/code-review-expert](https://github.com/sanyuan0704/code-review-expert) |
| `security-review` (community port) | Branch-level security review — auth, secrets, injection paths, deps. Pairs with the rubric's "Security" axis. | [skills.sh/zackkorman/skills/security-review](https://skills.sh/zackkorman/skills/security-review) | [github.com/zackkorman/skills](https://github.com/zackkorman/skills) |
| `react-best-practices` | TSX-aware reviewer — hooks, accessibility, performance, TS patterns. Triggers on multi-TSX-file diffs. | [skills.sh/vercel/vercel-plugin/react-best-practices](https://skills.sh/vercel/vercel-plugin/react-best-practices) | [github.com/vercel-labs/agent-skills](https://github.com/vercel-labs/agent-skills) |
| `accessibility-a11y` | WCAG checks for any UI-touching PR. Frontend Builder should self-check; Reviewer is the safety net. | [skills.sh/mindrally/skills/accessibility-a11y](https://skills.sh/mindrally/skills/accessibility-a11y) | [github.com/mindrally/skills](https://github.com/mindrally/skills) |
| `supabase-postgres-best-practices` | RLS, migration safety, index awareness — for any DB-touching PR. | [skills.sh/supabase/agent-skills/supabase-postgres-best-practices](https://skills.sh/supabase/agent-skills/supabase-postgres-best-practices) | [github.com/supabase/agent-skills/tree/main/supabase-postgres-best-practices](https://github.com/supabase/agent-skills/tree/main/supabase-postgres-best-practices) |
| `claude-api` | Verify any Anthropic-SDK usage (caching, model versions, tool-use shape). Triggers on diffs that import `anthropic`/`@anthropic-ai/sdk`. | [skills.sh/anthropics/skills/claude-api](https://skills.sh/anthropics/skills/claude-api) | [github.com/anthropics/skills/tree/main/claude-api](https://github.com/anthropics/skills/tree/main/claude-api) |
| `codeql` (Trail of Bits) | Static analysis review — pull from the security-relevant Trail of Bits subset. | [skills.sh/trailofbits/skills/codeql](https://skills.sh/trailofbits/skills/codeql) | [github.com/trailofbits/skills](https://github.com/trailofbits/skills) |
| `semgrep` (Trail of Bits) | Pattern-based static analysis. | [skills.sh/trailofbits/skills/semgrep](https://skills.sh/trailofbits/skills/semgrep) | [github.com/trailofbits/skills](https://github.com/trailofbits/skills) |
| `semgrep-rule-creator` (Trail of Bits) | Author Semgrep rules tuned to this workspace's patterns. | [skills.sh/trailofbits/skills/semgrep-rule-creator](https://skills.sh/trailofbits/skills/semgrep-rule-creator) | [github.com/trailofbits/skills](https://github.com/trailofbits/skills) |
| `variant-analysis` (Trail of Bits) | Find variants of a known bug across the codebase. | [skills.sh/trailofbits/skills/variant-analysis](https://skills.sh/trailofbits/skills/variant-analysis) | [github.com/trailofbits/skills](https://github.com/trailofbits/skills) |
| `differential-review` (Trail of Bits) | Diff-aware security review — focuses scrutiny on what changed. | [skills.sh/trailofbits/skills/differential-review](https://skills.sh/trailofbits/skills/differential-review) | [github.com/trailofbits/skills](https://github.com/trailofbits/skills) |
| `audit-context-building` (Trail of Bits) | Build the context an auditor needs before reviewing. | [skills.sh/trailofbits/skills/audit-context-building](https://skills.sh/trailofbits/skills/audit-context-building) | [github.com/trailofbits/skills](https://github.com/trailofbits/skills) |
| `fix-review` (Trail of Bits) | Verify a proposed fix actually addresses the root cause. | [skills.sh/trailofbits/skills/fix-review](https://skills.sh/trailofbits/skills/fix-review) | [github.com/trailofbits/skills](https://github.com/trailofbits/skills) |
| `sentry-for-ai` | Pull recent errors in touched paths. Lets Reviewer say: *"this file has 200 prod errors in the last 7 days — please add a regression test for case X"*. | [skills.sh/getsentry/sentry-for-ai](https://skills.sh/getsentry/sentry-for-ai) | [github.com/getsentry/sentry-for-ai](https://github.com/getsentry/sentry-for-ai) |
| `stripe-best-practices` | Only for PRs that touch billing. Webhooks, idempotency, currency handling — common review-trap territory. | [skills.sh/stripe/ai/stripe-best-practices](https://skills.sh/stripe/ai/stripe-best-practices) | [github.com/stripe/ai](https://github.com/stripe/ai) |
| `better-auth-security-best-practices` | Only if better-auth is the workspace's auth — security-focused review pass. | [skills.sh/better-auth/skills/better-auth-security-best-practices](https://skills.sh/better-auth/skills/better-auth-security-best-practices) | [github.com/better-auth/skills](https://github.com/better-auth/skills) |
| `react-native-best-practices` | Only if the workspace ships RN — performance traps Frontend Builder may miss. | [skills.sh/callstackincubator/agent-skills/react-native-best-practices](https://skills.sh/callstackincubator/agent-skills/react-native-best-practices) | [github.com/callstackincubator/agent-skills](https://github.com/callstackincubator/agent-skills) |

### To author via `skill-creator`

| Skill | Purpose |
|---|---|
| `reviewer-rubric` | Encodes the 5-axis rubric (correctness, security, tests, conventions, rollback-safety) so the model applies it consistently. Output template forces a top-of-PR scorecard. |
| `workspace-conventions-check` | Reads `workspace.context` and emits a checklist the Reviewer applies to each PR. Without this, conventions enforcement drifts. |
| `regression-risk` | Heuristics for "what could this PR break elsewhere?" — touches pages with high prod traffic, modifies a function with N call-sites, changes a public API. Pair with the Sentry skill. |

**CodeRabbit MCP (not a skill)**: `bradthebeeble/coderabbitai-mcp` is an MCP, not a `SKILL.md` package. Wire it as an MCP if CodeRabbit is also on the repo (see MCP table below); the agent reads CodeRabbit's review as a second-opinion checker.

## MCP servers (`agent.mcp_config`)

| MCP | Scope | Why |
|---|---|---|
| `github` | **READ-ONLY** for code (`contents:read`, `metadata:read`, `pull_requests:read`); **WRITE only for review comments** (`pull_requests:write`). **Explicitly NO** `repo` write, no `contents:write`, no `merge`, no `workflows`. | Read PR diff, post line-level comments, post review verdict. Cannot push, cannot merge. |
| `sentry` | read-only — projects scoped to the current workspace | Recent errors in touched paths/files. |
| `filesystem` | **READ-ONLY** scoped to the task `work_dir` only | Read the cloned repo for context (workspace conventions, sibling files, import graph). No writes. |
| `context7` | read-only | Verify Builder's library usage against current docs. |
| `multica` | scoped: file new issues, comment on issues, list workspace context — **no agent CRUD**, no skill CRUD | When Reviewer spots an architectural concern out-of-scope for the PR, it files a separate issue rather than bloating the PR thread. |
| *(optional)* `coderabbitai` | read-only against the same PR | If CodeRabbit is also wired to the repo, Reviewer can read its review and reconcile findings — diversity of evaluators. |

### How to wire this in Multica (Codex CLI)

Reviewer is the only Codex-runtime agent in this roster. Wiring is **fundamentally different** from the Claude Code agents because:

- Codex CLI has no `--mcp-config` flag. It reads MCP from `$CODEX_HOME/config.toml` using `[mcp_servers.NAME]` TOML tables.
- Multica's daemon does not yet inject `agent.mcp_config` into Codex's `config.toml` — [PR #1288](https://github.com/multica-ai/multica/pull/1288) is open as of April 2026 with reviewer pushback. Setting `mcp_config` on a Codex-runtime agent today is silently ignored.

The canonical TOML config lives at `agents/reviewer/mcp.toml` (sibling file, **not** `mcp.json` — Codex is TOML-only). Wire it up via one of:

1. **Pre-populate the daemon host's `~/.codex/config.toml`** (simplest while only one Codex agent exists). Copy `mcp.toml` to that path on the daemon machine, **inline real secrets** (Codex does not expand `${VAR}`), and `chmod 600`.
2. **Per-agent `CODEX_HOME` via wrapper script** (when adding a second Codex agent):
   - Wrapper: `export CODEX_HOME="$HOME/.multica/codex-homes/reviewer"; exec /usr/local/bin/codex "$@"`
   - Daemon env: `MULTICA_CODEX_PATH=/usr/local/bin/codex-reviewer`
   - Place `mcp.toml` at `~/.multica/codex-homes/reviewer/config.toml`.
3. **Multica GUI** (Settings → Agents → Reviewer):
   - **Custom arguments**: `--sandbox read-only --model gpt-5.1-codex`
   - **Environment**: not used for MCP secrets (Codex won't expand them). Setting `CODEX_HOME` here per-agent is a clean alternative to option 2.

Full procedure + workaround commentary in `agents/MCP-WIRING.md` Part B.

**Token hygiene** (non-negotiable):
- `GITHUB_PAT_REVIEWER` — fine-grained PAT distinct from the Builders'. Permissions: `Pull requests: read & write`, `Contents: read`, `Metadata: read`. **Not granted**: `Contents: write`, `Administration`, `Workflows`, `Actions: write`.
- `SENTRY_REVIEWER_TOKEN` — read-only org scope.
- Tokens are **inlined** in the daemon-host `config.toml` (TOML, no `${VAR}` expansion). Lock to mode 0600 and own by the daemon user.

## Coherence check

The package — Codex/Gemini runtime + read-only filesystem MCP + GitHub PAT with no merge/push scope + adversarial system prompt + no autoformatting nits — is what *prevents rubber-stamping*. Drop any one piece and the loop degrades: same-model = blind spots align; write-scoped GitHub token = Reviewer can accidentally push; missing rubric skill = inconsistent scoring across PRs. The whole point is that the Reviewer is *structurally incapable* of rubber-stamping, not just instructed not to.

### Sources
- `agent-roster.md` §5.4 (Reviewer spec), §7 (anti-pattern: same-model rubber-stamp).
- `agent-orchestration.md` §4.7 (5-axis evaluator rubric), §5 (anti-drift via model diversity).
- `multica.md` §3 (11 supported runtimes), §10 (per-agent runtime override env vars).
- Trail of Bits skills: https://github.com/trailofbits/skills
- CodeRabbit MCP: https://github.com/bradthebeeble/coderabbitai-mcp
- GitHub MCP: https://github.com/github/github-mcp-server
