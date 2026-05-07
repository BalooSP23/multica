# Skills + MCPs for Reviewer

## Skills

### Already installed locally (just attach)

> Note: Reviewer runs on Codex CLI, but Multica still imports its skills via `multica skill import --url <URL>` — the daemon writes them to `$CODEX_HOME/skills/<name>/` (`multica.md` §7). Same import flow as Claude Code agents. `review`, `security-review`, `simplify` ship with Claude Code; for a Codex-runtime agent, install them from skills.sh/GitHub since they aren't part of Codex's bundle.

| Skill | Source | skills.sh | clawhub.io | Why |
|---|---|---|---|---|
| `review` | bundled with Claude Code (Anthropic ships it locally; for Codex runtime, install via marketplace) | not on skills.sh as a standalone Anthropic skill — closest analogues: [skills.sh/wshobson/agents/code-review-excellence](https://skills.sh/wshobson/agents/code-review-excellence) and [skills.sh/sanyuan0704/code-review-expert/code-review-expert](https://skills.sh/sanyuan0704/code-review-expert/code-review-expert) | unreachable (cert/domain expired May 2026) — fallback for community equivalent: [github.com/wshobson/agents](https://github.com/wshobson/agents) | Core PR-review skill — reads diff, comments by line, summarizes verdict. The default trigger for any PR-review task. |
| `security-review` | bundled with Claude Code (community ports available) | community ports: [skills.sh/zackkorman/skills/security-review](https://skills.sh/zackkorman/skills/security-review) · [skills.sh/sickn33/antigravity-awesome-skills/security-review](https://skills.sh/sickn33/antigravity-awesome-skills/security-review) | unreachable — fallback: [github.com/zackkorman/skills](https://github.com/zackkorman/skills) | Branch-level security review — auth, secrets, injection paths, deps. Pairs with the rubric's "Security" axis. |
| `simplify` | bundled with Claude Code | not on skills.sh as a standalone (bundled in Claude Code) | n/a (bundled) | Catches over-engineering — dead code, premature abstraction, duplicated logic. Cheap signal that often catches the most reviewer-actionable items. |
| `vercel:react-best-practices` | `vercel/vercel-plugin` (also `vercel-labs/agent-skills`) | [skills.sh/vercel/vercel-plugin/react-best-practices](https://skills.sh/vercel/vercel-plugin/react-best-practices) (alt: [skills.sh/vercel-labs/agent-skills/vercel-react-best-practices](https://skills.sh/vercel-labs/agent-skills/vercel-react-best-practices)) | unreachable — fallback: [github.com/vercel-labs/agent-skills](https://github.com/vercel-labs/agent-skills) | TSX-aware reviewer — hooks, accessibility, performance, TS patterns. Triggers on multi-TSX-file diffs. |
| `accessibility-a11y` | `mindrally/skills` | [skills.sh/mindrally/skills/accessibility-a11y](https://skills.sh/mindrally/skills/accessibility-a11y) | unreachable — fallback: [github.com/mindrally/skills](https://github.com/mindrally/skills) | WCAG checks for any UI-touching PR. Frontend Builder should self-check; Reviewer is the safety net. |
| `supabase:supabase-postgres-best-practices` | `supabase/agent-skills` | [skills.sh/supabase/agent-skills/supabase-postgres-best-practices](https://skills.sh/supabase/agent-skills/supabase-postgres-best-practices) | unreachable — fallback: [github.com/supabase/agent-skills/tree/main/supabase-postgres-best-practices](https://github.com/supabase/agent-skills/tree/main/supabase-postgres-best-practices) | RLS, migration safety, index awareness — for any DB-touching PR. |
| `claude-api` | `anthropics/skills/claude-api` | [skills.sh/anthropics/skills/claude-api](https://skills.sh/anthropics/skills/claude-api) | unreachable — fallback: [github.com/anthropics/skills/tree/main/claude-api](https://github.com/anthropics/skills/tree/main/claude-api) | Verify any Anthropic-SDK usage (caching, model versions, tool-use shape) against current API surface. Triggers on diffs that import `anthropic`/`@anthropic-ai/sdk`. |

### Needs installation
| Skill | Source | skills.sh | clawhub.io | Why |
|---|---|---|---|---|
| **Trail of Bits security skill set** (50 skills total; pick the security-relevant subset: `codeql`, `semgrep`, `semgrep-rule-creator`, `variant-analysis`, `differential-review`, `audit-context-building`, `fix-review`) | `trailofbits/skills` (CC BY-SA 4.0) | publisher index: [skills.sh/trailofbits/skills](https://skills.sh/trailofbits/skills) — individual: [codeql](https://skills.sh/trailofbits/skills/codeql) · [semgrep](https://skills.sh/trailofbits/skills/semgrep) · [semgrep-rule-creator](https://skills.sh/trailofbits/skills/semgrep-rule-creator) · [variant-analysis](https://skills.sh/trailofbits/skills/variant-analysis) · [differential-review](https://skills.sh/trailofbits/skills/differential-review) · [audit-context-building](https://skills.sh/trailofbits/skills/audit-context-building) · [fix-review](https://skills.sh/trailofbits/skills/fix-review) | unreachable (cert/domain expired May 2026) — fallback: [github.com/trailofbits/skills](https://github.com/trailofbits/skills) | The gold-standard security audit pack — CodeQL, Semgrep, variant analysis, differential code review, audit-context building. Installs as a marketplace; pick the subset relevant to the workspace's stack. Cite: `skills-bank.md` §6 "Security". *Note: Trail of Bits ships `differential-review` and `fix-review` (not `differential-code-review` / `fix-verification` as previously documented).* |
| **CodeRabbit review skill** | `bradthebeeble/coderabbitai-mcp` (MCP, not a SKILL.md package) | not published as a skill on skills.sh | unreachable — fallback: [github.com/bradthebeeble/coderabbitai-mcp](https://github.com/bradthebeeble/coderabbitai-mcp) | Encodes a published, well-tuned review rubric. Run alongside the local `review` skill as a second-opinion checker; if both flag the same line, confidence is high. Cite: `skills-bank.md` §6 "Observability & review". |
| **Sentry skill** | `getsentry/sentry-for-ai` (31 skills) | publisher index: [skills.sh/getsentry/sentry-for-ai](https://skills.sh/getsentry/sentry-for-ai) (also CLI: [skills.sh/sentry/dev/sentry-cli](https://skills.sh/sentry/dev/sentry-cli)) | unreachable — fallback: [github.com/getsentry/sentry-for-ai](https://github.com/getsentry/sentry-for-ai) | Pull recent errors in the touched paths. Lets Reviewer say: *"this file has 200 prod errors in the last 7 days — please add a regression test for case X"*. |
| **stripe/stripe-best-practices** | `stripe/ai` | [skills.sh/stripe/ai/stripe-best-practices](https://skills.sh/stripe/ai/stripe-best-practices) | unreachable — fallback: [github.com/stripe/ai/tree/main/skills/stripe-best-practices](https://github.com/stripe/ai/tree/main/skills/stripe-best-practices) | Only for PRs that touch billing. Webhooks, idempotency, currency handling — common review-trap territory. |
| **better-auth/best-practices** | `better-auth/skills` | [skills.sh/better-auth/skills/better-auth-best-practices](https://skills.sh/better-auth/skills/better-auth-best-practices) (security-only variant: [better-auth-security-best-practices](https://skills.sh/better-auth/skills/better-auth-security-best-practices)) | unreachable — fallback: [github.com/better-auth/skills](https://github.com/better-auth/skills) | Only if better-auth is the workspace's auth — review-time sanity check. |
| **callstackincubator/react-native-best-practices** | `callstackincubator/agent-skills` | [skills.sh/callstackincubator/agent-skills/react-native-best-practices](https://skills.sh/callstackincubator/agent-skills/react-native-best-practices) | unreachable — fallback: [github.com/callstackincubator/agent-skills](https://github.com/callstackincubator/agent-skills) | Only if the workspace ships RN — performance traps Frontend Builder may miss. |

### To author via `skill-creator`
| Skill | Purpose |
|---|---|
| `reviewer-rubric` | Encode the 5-axis rubric (correctness, security, tests, conventions, rollback-safety) so the model applies it consistently across reviews. Output template forces a top-of-PR scorecard. |
| `workspace-conventions-check` | Reads `workspace.context` for the workspace and emits a checklist the Reviewer applies to each PR. Without this, conventions enforcement drifts. |
| `regression-risk` | Heuristics for "what could this PR break elsewhere?" — touches pages with high prod traffic, modifies a function with N call-sites, changes a public API. Pair with Sentry skill. |

## MCP servers (`agent.mcp_config`)

| MCP | Scope | Why |
|---|---|---|
| `github` | **READ-ONLY** for code (`contents:read`, `metadata:read`, `pull_requests:read`); **WRITE only for review comments** (`pull_requests:write`). **Explicitly NO** `repo` write, no `contents:write`, no `merge`, no `workflows`. | Read PR diff, post line-level comments, post review verdict. Cannot push, cannot merge, cannot edit other users' comments (the GitHub permission model handles that). |
| `sentry` | read-only — projects scoped to the current workspace | Recent errors in touched paths/files; "this file has prod errors → add a regression test before merging". |
| `filesystem` | **READ-ONLY** scoped to the task `work_dir` only | Read the cloned repo for context (workspace conventions, sibling files, import graph). No writes. |
| `context7` | read-only | Verify Builder's library usage against current docs — especially when Builder upgrades a dep or uses an API the Reviewer's training data is stale on. |
| `multica` | scoped: file new issues, comment on issues, list workspace context — **no agent CRUD**, no skill CRUD | When Reviewer spots an architectural concern that's out-of-scope for the PR, it files a separate issue rather than bloating the PR thread. |
| *(optional)* `coderabbitai` | read-only against the same PR | If CodeRabbit is also wired to the repo, Reviewer can read its review and reconcile findings — diversity of evaluators. |

### How to wire this in Multica (Codex CLI, no UI, no daemon-side injection yet)

Reviewer is the only Codex-runtime agent in this roster. Wiring is **fundamentally different** from the Claude Code agents because:

- Codex CLI has no `--mcp-config` flag. It reads MCP from `$CODEX_HOME/config.toml` (default `~/.codex/config.toml`) using `[mcp_servers.NAME]` TOML tables.
- Multica's daemon does not yet inject `agent.mcp_config` into Codex's config.toml — [PR #1288](https://github.com/multica-ai/multica/pull/1288) is open as of April 2026 with reviewer pushback. Setting `mcp_config` on a Codex-runtime agent today is silently ignored.

The canonical TOML config lives at `agents/reviewer/mcp.toml` (sibling file, **not** `mcp.json` — Codex is TOML-only). Wire it up via one of:

1. **Pre-populate the daemon host's `~/.codex/config.toml`** (simplest while only one Codex agent exists). Copy `mcp.toml` to that path on the daemon machine, **inline real secrets** (Codex does not expand `${VAR}` syntax), and `chmod 600`.

2. **Per-agent `CODEX_HOME` via wrapper script** (when adding a second Codex agent):
   - Wrapper: `export CODEX_HOME="$HOME/.multica/codex-homes/reviewer"; exec /usr/local/bin/codex "$@"`
   - Daemon env: `MULTICA_CODEX_PATH=/usr/local/bin/codex-reviewer`
   - Place `mcp.toml` at `~/.multica/codex-homes/reviewer/config.toml`.

3. **Multica GUI** (Settings → Agents → Reviewer):
   - **Custom arguments**: `--sandbox read-only --model gpt-5.1-codex` (or whatever model + sandbox you want)
   - **Environment**: not used for MCP secrets today (Codex won't expand them) — leave empty unless setting `CODEX_HOME` directly here. If `CODEX_HOME` is set per-agent in Environment, the daemon-host `~/.codex/config.toml` is bypassed and the per-agent dir is used instead. This is cleaner than option 2 and avoids the wrapper script.

Full procedure + workaround commentary lives in `agents/MCP-WIRING.md` Part B.

**Token hygiene (non-negotiable)**:
- `GITHUB_PAT_REVIEWER` is a fine-grained PAT distinct from the Builders'. Permissions: `Pull requests: read & write`, `Contents: read`, `Metadata: read`. **Not granted**: `Contents: write`, `Administration`, `Workflows`, `Actions: write`. Branch-protection on `main` should require human approval regardless — defense in depth.
- `SENTRY_REVIEWER_TOKEN` is read-only org scope.
- Tokens are **inlined** in the daemon-host `config.toml` (TOML, no `${VAR}` expansion). Lock the file to mode 0600 and own it by the daemon user. Rotation is a daemon-host file edit, not an agent-record edit.

## Coherence check

The package — Codex/Gemini runtime + read-only filesystem MCP + GitHub PAT with no merge/push scope + adversarial system prompt + no autoformatting nits — is what *prevents rubber-stamping*. Drop any one piece and the loop degrades: same-model = blind spots align; write-scoped GitHub token = Reviewer can accidentally push; missing rubric skill = inconsistent scoring across PRs. The whole point is that the Reviewer is *structurally incapable* of rubber-stamping, not just instructed not to. Hard rules in the system prompt are the last line of defense, not the first.

---

**Sources cited**:
- `agent-roster.md` §5.4 (Reviewer spec), §7 (anti-pattern: same-model rubber-stamp).
- `agent-orchestration.md` §4.7 (5-axis evaluator rubric), §5 (anti-drift via model diversity).
- `multica.md` §3 (11 supported runtimes), §10 (per-agent runtime override env vars).
- `skills-bank.md` §6 (Trail of Bits, CodeRabbit, Sentry, security-review).
- Anthropic, *Building Effective Agents* — evaluator-optimizer pattern: https://www.anthropic.com/research/building-effective-agents
- Trail of Bits skills: https://github.com/trailofbits/skills
- CodeRabbit MCP: https://github.com/bradthebeeble/coderabbitai-mcp
- GitHub MCP `pull_request_review_write` capabilities: https://github.com/github/github-mcp-server
