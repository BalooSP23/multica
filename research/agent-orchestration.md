# Agent Orchestration — Best Practices

> Synthesis of Anthropic's official agent guidance (Building Effective Agents, Multi-agent Research System engineering blog, anthropic-cookbook patterns) and 2026 community consensus on multi-agent frameworks. The goal: a working theory of *when to add agents, what pattern to use, and how not to set token bills on fire*.

---

## TL;DR — three rules that beat 90% of mistakes

1. **Start with the simplest thing that works.** A single Augmented LLM with the right tools beats most "agent systems" you'd design over a weekend. Add complexity only after measuring a concrete bottleneck.
2. **Multi-agent systems cost ~15× more tokens than a single chat.** Anthropic measured this on real research workloads. Multi-agent is justified only when output value > 15× the marginal cost.
3. **Sub-agents fail in predictable ways unless you brief them in four parts: objective, output format, tool guidance, task boundaries.** Vague prompts cause duplicate work and gaps. Detailed prompts cut research time up to 90%.

Skip to §10 for the framework grid; §11 for how each pattern composes onto Multica primitives.

---

## 1. The decision tree — do you actually need multiple agents?

```
Is the task open-ended? ─── no ──▶ workflow (deterministic chain)
                                      │
                                      ▼
                                  prompt chaining or routing

Yes, open-ended.
   │
   ▼
Is the value of the answer > ~15× the cost of a single chat? ─── no ──▶ single Augmented LLM
   │
   ▼ yes
Are subtasks independent, or does each step depend on the previous? 
   │
   ├─ independent ─────────▶ parallelization
   │
   └─ dependent  ──────────▶ orchestrator-workers
                                 │
                                 ▼
                             Need iteration on quality? ── yes ──▶ wrap with evaluator-optimizer
```

**When *not* to multi-agent**:
- Tasks that need shared state across all agents (most coding work).
- Tasks where a single context window comfortably holds everything.
- Tasks where latency matters more than completeness.
- Anything you haven't first tried with one well-tooled agent.

**When multi-agent earns its keep** (Anthropic's research-system numbers):
- Lead-Opus + Sonnet sub-agents beat single-Opus by **+90.2%** on open-ended research tasks.
- Three factors explain ~95% of variance in quality: **token usage (~80%), tool-call count, model choice**. Spending more tokens on a smarter model is the highest-leverage knob.

---

## 2. The building block — Augmented LLM

Every higher pattern composes this primitive:

```
LLM ── retrieval ── tools ── memory ── (optionally: structured output)
```

What "augmented" means in practice:
- The model **chooses** when to retrieve, which tool to call, and what to remember — it isn't routed externally.
- Tools should return concise, focused, paginatable data.
- Memory is everything across turns: prior plans, intermediate artifacts, learned constraints.
- Structured output (JSON schema, tool calls) is what lets you compose this primitive with deterministic code.

**Rule**: add a tool before adding an agent. Most "I need an agent" problems are really "I need a better tool".

---

## 3. The seven canonical patterns (Anthropic, *Building Effective Agents*)

For each: definition · ASCII shape · use when · don't use · failure mode · debug.

### 3.1 Prompt chaining

**Sequential, fixed steps. Each call processes the previous output.**

```
[input] → LLM₁ → [intermediate] → LLM₂ → [output]
```

- *Use when*: the task **decomposes cleanly** into known stages (outline → draft → polish; classify → extract → format).
- *Don't use*: when you don't know the steps in advance, or steps have to retry.
- *Failure mode*: one stage's failure poisons the chain; total latency = sum of stages.
- *Debug*: log each intermediate output; the last good output tells you which stage broke.

### 3.2 Routing

**Classifier sends each input to exactly one specialist.**

```
[input] → router LLM ──┬──▶ specialist A
                       ├──▶ specialist B
                       └──▶ specialist C
```

- *Use when*: input categories are **distinct** and each deserves a tailored prompt (support tickets by topic, code questions by language, multilingual content).
- *Don't use*: when an input genuinely needs multiple specialists at once.
- *Failure mode*: misclassification sends work to the wrong specialist; cross-domain inputs degrade.
- *Debug*: log router decisions, compare against expected categories, check confidence margin.

### 3.3 Parallelization (sectioning)

**Split independent subtasks, run concurrently, combine.**

```
                ┌──▶ LLM (section A) ──┐
[input] split ──┼──▶ LLM (section B) ──┼─▶ merge ─▶ [output]
                └──▶ LLM (section C) ──┘
```

- *Use when*: subtasks **don't depend on each other** (review code for security AND performance AND style; summarize each chapter of a doc).
- *Don't use*: when sections need to know about each other.
- *Failure mode*: sections drift in style; merge step has to reconcile.
- *Debug*: compare sections side-by-side; if one is consistently low quality, its prompt is the problem.

### 3.4 Parallelization (voting / sampling)

**Same task N times, aggregate via voting / averaging / best-of-N.**

```
[input] ──▶ LLM run 1 ──┐
[input] ──▶ LLM run 2 ──┼─▶ aggregate ─▶ [output]
[input] ──▶ LLM run 3 ──┘
```

- *Use when*: **multiple perspectives improve confidence** (guardrails, factual claims, code generation where you'll pick the best).
- *Don't use*: when you can't tell good from bad output cheaply.
- *Failure mode*: all N calls share the same blind spot.
- *Debug*: vary temperature / model across runs; if all agree it's *probably* right but might be confidently wrong.

### 3.5 Orchestrator-workers

**Central LLM dynamically decomposes the task and dispatches workers.**

```
[input] → orchestrator ┬──▶ worker (subtask 1) ──▶ artifact 1
                       ├──▶ worker (subtask 2) ──▶ artifact 2  
                       └──▶ worker (subtask 3) ──▶ artifact 3
                                  │
                       merger / synthesizer ──▶ [output]
```

- *Use when*: you **can't predict the subtasks in advance** — they emerge from the input. (Research, multi-file code edits, complex investigations.)
- *Don't use*: when the decomposition is fixed (use sectioning instead).
- *Failure mode*: orchestrator becomes a bottleneck; runaway spawning (50 sub-agents for a small query); duplicate work across workers; over-broad task descriptions.
- *Debug*: log the orchestrator's decomposition explicitly; verify worker scopes are non-overlapping; cap sub-agent count.

This is "the most powerful and most common pattern in production" (heyuan110.com 2026 taxonomy).

### 3.6 Evaluator-optimizer

**Generator produces output; evaluator scores and feeds back; generator iterates.**

```
[input] → generator → [draft] → evaluator → score + feedback
                          ▲                       │
                          └───── if not good ─────┘
```

- *Use when*: criteria are **clear** and refinement compounds (code that has to compile, content with a style guide, structured outputs that must validate).
- *Don't use*: when evaluation is subjective or expensive; when criteria are squishy.
- *Failure mode*: infinite loops; adversarial drift between gen and eval; latency stacks.
- *Debug*: hard-cap iteration count; log score trajectory (should be monotonic); if not, the evaluator is unreliable.

### 3.7 Autonomous agent (open loop)

**LLM in a tool-use loop, advancing on environment feedback, no fixed step count.**

```
loop:
   observe environment  →  decide action  →  call tool  →  read result
                                                  │
                                                  ▼
                                             stop condition?
```

- *Use when*: open-ended problems where the **number of steps is genuinely unknowable**.
- *Don't use*: anywhere a workflow would do. Workflows are cheaper, more predictable, and easier to debug.
- *Failure mode*: runaway loops; getting stuck on rabbit holes; over-search past sufficiency.
- *Debug*: enforce step caps, time caps, token caps; require explicit stop signals; instrument tool calls to detect repetition.

---

## 4. Cross-pattern best practices (from Anthropic's multi-agent engineering writeup)

These are the practices that tend to matter regardless of which pattern you pick.

### 4.1 Sub-agent prompt anatomy

Every sub-agent invocation **must** carry these four pieces:

| Piece | What it answers | Bad example | Good example |
|---|---|---|---|
| **Objective** | What does success look like? | "research X" | "Identify the 3 most-cited 2026 papers on X and extract their key claims" |
| **Output format** | How should you return it? | "summarize" | "Return JSON: `[{title, authors, year, key_claim, citation_count}]`" |
| **Tool guidance** | Which tools, in what order? | (silent) | "Prefer `arxiv_search` for primary, `web_search` only as fallback" |
| **Task boundaries** | What is *not* yours? | (silent) | "Do not investigate sibling topic Y; do not summarize beyond key_claim" |

Anthropic's finding: vague briefs caused sub-agents to duplicate searches, leave gaps, or wander. Detailed briefs cut research time by **up to 90%**.

### 4.2 Parallelism that actually pays off

- **3–5 sub-agents in parallel** at the orchestrator level.
- **Each sub-agent uses 3+ tools in parallel** within its turn.
- Past 5 concurrent sub-agents you're paying for marginal coverage you usually don't need.

### 4.3 Artifact handoff over chat-based handoff

The "telephone game" failure mode: sub-agent A summarizes a finding to the orchestrator, the orchestrator summarizes it to sub-agent B, B drops half the nuance.

**Mitigation**: sub-agents write outputs to the **filesystem**. Only the *reference* (a path or ID) is passed in chat. The next agent reads the artifact directly.

In Multica terms: this is exactly why every task has its own `work_dir` and skills are written as files.

### 4.4 Memory checkpointing

Claude's 200k context window fills faster than you think on long-horizon tasks. Before saturation:
- Summarize completed phases and write the summary to external memory (a file, a DB row).
- Drop intermediate artifacts from the active context once the summary is good.
- New phases load only the summary plus what they specifically need.

### 4.5 Interleaved / extended thinking as a controllable scratchpad

Visible "thinking" between tool calls is a planning surface, not just internal monologue. Use it as a **budget knob**: more thinking on hard subtasks, none on routine ones. Anthropic's research system uses interleaved thinking to evaluate tool results and refine queries mid-task.

### 4.6 End-state evals over step-by-step evals

Multi-agent runs are non-deterministic. Don't try to assert "step 3 must produce output X". Instead: define **discrete checkpoints** at which specific state must hold (file written, JSON validates, test passes), and evaluate the **final state** against the rubric.

### 4.7 LLM-as-judge with a five-axis rubric

Common rubric for research/synthesis tasks:
1. **Factual accuracy** — claims match cited sources.
2. **Citation accuracy** — sources actually say what's claimed.
3. **Completeness** — did it cover the brief?
4. **Source quality** — primary > secondary > SEO content farm.
5. **Tool efficiency** — minimal redundant calls.

A single LLM call returning floats `0.0–1.0` per axis is enough to bootstrap. You'll catch ~30% → 80% improvements with as few as **20 representative test queries**.

### 4.8 Rainbow deployments for prompt rollouts

When you change a sub-agent prompt that's actively running, you don't want to interrupt in-flight tasks. Gradually shift traffic from prompt-vN to prompt-vN+1, watching the eval rubric. If it regresses, roll back.

### 4.9 Human exploration

Automated evals find regressions; humans find new failure modes (hallucinations, bias, weird edge cases). Both, not either.

---

## 5. Failure mode catalogue — and one mitigation each

| Failure | Symptom | Mitigation |
|---|---|---|
| Runaway spawning | Orchestrator launches 50 sub-agents for a 1-line query | Hard-cap sub-agent count in the orchestrator prompt; reject task decompositions above N |
| Duplicated work | Multiple sub-agents do the same search | Explicit task boundaries in each sub-agent brief; orchestrator marks a "not yours" list |
| Gaps | Sub-tasks together don't cover the brief | Orchestrator emits a coverage manifest before dispatch; merger checks against it |
| Source bias | Sub-agents prefer SEO content over authoritative | Tool guidance: name preferred sources; demote known low-quality domains |
| Token sprawl | Cost spirals as sub-agents over-search | Effort budget per sub-agent ("at most 5 tool calls; stop when sufficient"); orchestrator enforces |
| "Telephone" loss | Information degrades hop by hop | Artifact handoff (filesystem references), not chat summaries |
| Single-orchestrator chokepoint | Orchestrator stalls = whole system stalls | Make orchestrator stateless and idempotent; checkpoint after each dispatch round |
| Infinite eval loops | Generator + evaluator never converge | Hard iteration cap; detect monotonic-non-improvement and exit with best-so-far |
| Context saturation | 200k ceiling hit mid-run | Phase summaries written to memory before the run grows past ~150k |
| Adversarial gen/eval drift | Generator games the evaluator | Vary evaluator model between iterations; use an independent test set for final acceptance |

---

## 6. The cheapest-implementation rule

```
tools  →  agent  →  framework  →  custom orchestration
```

**Cutover signals** (move one step right only when these are real):

| From → To | Signal |
|---|---|
| Tool → Agent | "I'm calling the same tool 5+ times manually per task" |
| Single agent → Multi-agent workflow | "One agent's context blows past 100k on a normal request" or "I want to run subtasks in parallel" |
| Workflow → Orchestrator-workers | "I can't predict the subtasks before seeing the input" |
| Orchestrator-workers → Custom framework | Measured, repeated bottleneck specific frameworks don't address (rare) |

The naive failure: starting at "custom multi-agent framework with 7 specialist roles" because it sounds powerful. You burn a week on plumbing and then discover one well-tooled agent solved the problem.

---

## 7. Framework selection grid (2026 landscape)

| Framework | Layer | Control flow | State | Native handoff | Parallelism | Persistence | Choose when |
|---|---|---|---|---|---|---|---|
| **LangGraph** | Single-agent / single-app engine | Directed graph + conditional edges | Checkpointed graph state | Edges in graph | Yes (concurrent nodes) | Built-in checkpoints | You need explicit, debuggable control flow with replay |
| **CrewAI** | Single-app engine | Linear / hierarchical (Process types) | Shared crew memory | Implicit (sequential) | Limited | Replay mechanism | You have a small fixed crew of well-defined roles |
| **AutoGen** | Single-app engine | Group chat + selector agent | Conversation log | Selector picks next speaker | Yes (concurrent groups) | Conversation history | Agents converging via dialogue |
| **OpenAI Swarm / Agents SDK** | Single-app engine | Explicit handoffs | Conversation | First-class handoff primitive | Implicit per branch | Conversation | Decentralized routing, hand-offs are the abstraction |
| **Claude Agent SDK** | Single-agent engine + sub-agents | Tool-use loop + sub-agent spawning + hooks | Filesystem + session | Sub-agents | Yes (parallel sub-agents) | Filesystem + sessions | Native to the Claude/Anthropic stack; want skills, hooks, plugins |
| **Multica** | **Coordination above agent CLIs** | Issue assignment + Autopilot triggers | Postgres + per-task work_dir | `@`-mention reassigns | Multiple tasks across daemons | Full database persistence | You have ≥2 humans + agents needing a shared backlog with audit trail |

The framework families don't compete with Multica. Multica drives any of them: the agent CLI Multica spawns can itself be running LangGraph-style sub-agents, CrewAI crews, or Claude Agent SDK code.

---

## 8. The 15× number, in context

Anthropic, on real research workloads:

> A multi-agent system with Claude Opus 4 as the lead agent and Claude Sonnet 4 sub-agents outperformed single-agent Claude Opus 4 by 90.2%. […] Multi-agent systems consume roughly 15× more tokens than single-agent chat interactions.

The ratio `15×` is the **starting cost-of-multi-agent** before you do anything clever. The number worth knowing alongside it: **agents typically use ~4× more tokens than chat interactions**. So multi-agent is ~3.75× more than a single agent.

If your task's value-per-completion isn't comfortably above 15× a single chat, **don't multi-agent it**.

---

## 9. Evaluation sketch (the minimum viable)

If you skip evaluation you'll regress without knowing it.

**Setup**:
1. Pick **20 representative test queries**. Spread across easy / medium / hard, common / edge.
2. Define a 5-axis rubric (see §4.7).
3. Pick an **independent** judge model (different from generator).
4. Generate, judge, log scores per axis per query.

**Useful aggregations**:
- Pass rate per axis.
- Worst-5 queries (read them; they show you the failure modes).
- Cost per query (tokens × $) — multi-agent regression checks.
- Time per query.

**Iteration loop**:
- Change one thing at a time (prompt, tool, model).
- Re-run the 20.
- Diff the rubric.

You'll see 30% → 80% jumps from prompt changes early; gains shrink fast. Stop iterating when the jump is below your noise floor (~5%).

---

## 10. Composing patterns onto Multica primitives

Multica ships with primitives that map almost 1-to-1 onto the canonical patterns. If you self-host Multica (see `multica.md`), you get most of these for free.

| Pattern | Multica primitive | How to wire it |
|---|---|---|
| **Routing** | Multiple agents in a workspace, distinct profiles | Assign issues to the agent whose role matches; or `@`-mention to switch agents mid-thread |
| **Prompt chaining** | Agent A closes issue, Autopilot creates next issue, agent B picks up | Or just one agent following a SKILL.md that prescribes the chain |
| **Parallelization (sectioning)** | Multiple concurrent tasks on different issues | Set `MULTICA_DAEMON_MAX_CONCURRENT_TASKS` high enough; scope issues so they don't fight over files |
| **Parallelization (voting)** | Two agents on the same issue, you pick the better PR | Or one Autopilot fan-out with concurrency policy `queue` to merge |
| **Orchestrator-workers** | Orchestrator agent comments on parent issue with `@worker create child issue: …`, workers claim children | Orchestrator gets a SKILL.md teaching it to decompose + delegate |
| **Evaluator-optimizer** | Two agents on one issue: builder pushes commits, reviewer agent triggered by `@reviewer please check` | Optionally automated via Autopilot on PR-open webhook |
| **Memory** | Workspace Context (system-prompt-for-all) + per-agent skills + session resumption | Same `(agent, issue)` reuses `session_id` and `work_dir` — long-horizon work without context blow-out |
| **Artifact handoff** | The task's `work_dir` is the filesystem artifact; sub-tasks reuse it via session resumption | Don't summarize between agents — point them at the directory |
| **Rainbow deployments** | Roll out a new SKILL.md at the workspace level; only **new** tasks pick it up; in-flight tasks finish on the old version | The skills design explicitly bakes this in |

The pattern that benefits most from Multica: **orchestrator-workers**. Without Multica you wire it manually (sub-process spawn, IPC, status reconciliation). With Multica, the orchestrator literally creates child issues and assigns them — the daemon does dispatch, the database does state, the WebSocket does observability.

---

## 11. Sources

- Anthropic, *Building Effective Agents* — https://www.anthropic.com/research/building-effective-agents
- Anthropic Engineering, *How we built our multi-agent research system* — https://www.anthropic.com/engineering/multi-agent-research-system
- Anthropic Engineering, *Equipping agents for the real world with Agent Skills* — https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills
- Anthropic Cookbook, orchestrator-workers notebook — https://github.com/anthropics/anthropic-cookbook/blob/main/patterns/agents/orchestrator_workers.ipynb
- *Multi-Agent Orchestration: 4 Patterns That Actually Work* (2026 taxonomy) — https://www.heyuan110.com/posts/ai/2026-02-26-multi-agent-orchestration/
- *Best Multi-Agent Frameworks in 2026* — https://gurusup.com/blog/best-multi-agent-frameworks-2026
- *Agent Architecture Patterns: 2026 Taxonomy Guide* — https://www.digitalapplied.com/blog/agent-architecture-patterns-taxonomy-2026
- *2026 AI Agent Framework Showdown* — https://qubittool.com/blog/ai-agent-framework-comparison-2026
- Cross-references: `multica.md` (Multica primitives), `skills-bank.md` (curated skills, including orchestration-related skills like `skill-creator` and `mcp-builder`)
