# FIC for Brownfield Codebases — Composable 1–5 Day Mini-Projects (Nov 2025)

> Premise: In mature (“brownfield”) repos, agentic coding often **slows experts** and increases **rework/slop**; success hinges on **context engineering** and **task-scoped isolation**, not chatty prompting. ([arXiv][1])
> Preference: **Claude Agent SDK** (+ Claude Code sub-agents & Skills) for implementation. ([Anthropic][2])

---

## 0) Program Setup (Day 0.5–1)

* **0.1 Goals & guardrails.** Define north-star metrics: *TTFPR* (time-to-first-PR), *Rework%*, *Merge-without-change%*, *Reviewer load*. Rationale: empirical studies show slowdown & slop without process. ([arXiv][1])
* **0.2 Repo skeleton.** Create mono-repo `fic-kit/` (MIT) with `apps/cli`, `agents/`, `skills/`, `docs/`. Shipping makes the whole deliverable **acquirable** (clone/run).

---

## 1) SDK Wiring: Claude Agent “Worker” (1–2 days)

* **Scope.** Minimal **Agent SDK** project that can run tools: FS, shell, git, tests. Provide `fic run <task>`. ([Anthropic][2])
* **Deliverables.** `agents/worker/` with: tool registry, run loop, stop-on-diff policy, dry-run mode.
* **Value.** Baseline to plug all later modules.

---

## 2) Context Compaction Pipeline (Research → Plan) (2–4 days)

* **2.1 research.md generator.** Sub-agents explore code (grep/ctags/AST), return **dense** summary: key files, types, call-graph, data flow, build/test cmds. *No raw logs.* ([Anthropic][3])
* **2.2 plan.md generator.** Given `research.md`, produce stepwise plan: files to change, function-level diffs, verification per step.
* **2.3 headroom policy.** Enforce 40–60% token use; Compaction = default. (Context utility focus.) ([Anthropic][3])
* **Deliverables.** `agents/researcher/`, `agents/planner/`, schemas for both docs.
* **Value.** Eliminates “vibe coding” drift; creates reviewable, short specs before code.

---

## 3) Sub-Agent Framework (Task-Scoped Isolation) (1–3 days)

* **3.1 Ephemeral sub-agents.** Stand up per noisy task (search, summarization). **Return JSON only** (strict schema). Discard their context post-call. ([Claude Docs][4])
* **3.2 Schema contracts.** JSONSchema for `FindUsages`, `SummarizeFile`, `ListTests`; include provenance (paths, lines, SHAs).
* **Deliverables.** `agents/subagent/` runner, `schemas/*.json`.
* **Value.** Prevents context pollution; solves “playing telephone”.

---

## 4) “Ralph” Loop Utility (Stateless Stepper) (0.5–1 day)

* **Scope.** `ralph` bash/Node CLI: `while true; do agent <PROMPT.md>; done` with idempotent step keying & checkpoint files. ([devtalk.com][5])
* **Use.** Great for port/refactor sweeps where instructions are static.
* **Deliverable.** `apps/ralph/`.
* **Value.** Zero context accumulation; simple, reliable iteration.

---

## 5) Brownfield Code Index (Symbol-Aware RAG) (2–4 days)

* **Scope.** Build fast xref index (ctags/Tree-sitter). APIs: `search symbol`, `incoming/outgoing calls`, `test coverers`.
* **Deliverable.** `services/code-index/` with on-disk shards + CLI (`fic xref ...`).
* **Value.** Raises **Correctness & Completeness** while keeping Size minimal (dense facts > raw files). ([Anthropic][3])

---

## 6) Log/Trace Distillers (1–2 days)

* **Scope.** Parsers for build/test/runtime logs → compact, ranked facts (error locus, failing test, suspected commit).
* **Deliverable.** `services/log-distill/` with adapters (JUnit, pytest, Maven/Gradle, Go test).
* **Value.** Converts classic “context eaters” into high-density signal. ([Anthropic][3])

---

## 7) Token Budgeter & Perf Heuristic (0.5–1 day)

* **Scope.** Compute live budget and **Performance ∝ (Correctness²×Completeness)/Size** scorecard per task; gate runs if score low.
* **Deliverable.** `libs/context-metrics/` + CLI report.
* **Value.** Operationalizes context discipline.

---

## 8) Guardrails: Correctness-First Execution (2–3 days)

* **8.1 Step gates.** Every plan step = patch → compile/test → static checks; auto-revert on red.
* **8.2 Diff minimalism.** Enforce **“small, reviewable diffs”**; block wide refactors unless explicitly planned.
* **Deliverable.** `agents/executor/` with gates, `policies/diff.yml`.
* **Value.** Tackles subtle bugs & “tech-debt factory” failure mode flagged by practitioners & experiments. ([arXiv][1])

---

## 9) PR Generator + Safety (1–2 days)

* **Scope.** Turn finished plan phases into **stacked PRs**; auto-label provenance (files, tests run, failing tests fixed).
* **Deliverable.** `integrations/github/` or GitLab runner; PR templates.
* **Value.** Human-legible outputs; easier merge & rollback.

---

## 10) Mental-Alignment Review Pack (MAR) (1–2 days)

* **Scope.** Render `research.md` + `plan.md` + coverage map; reviewers comment **specs first** (not code first).
* **Deliverable.** Static site (`docs/mar/`) with deep-links to code lines.
* **Value.** Code review’s top job is **team mental alignment**, not mere bug-finding. ([Blake Smith][6])

---

## 11) Slop/Rework Telemetry (2–4 days)

* **11.1 CCP/Corrective-commit.** Track **Corrective Commit Probability** per team/repo to quantify rework. ([arXiv][7])
* **11.2 “Workslop” detectors.** Heuristics: refactor churn without issue linkage; PRs reverted within N days; review load spikes. Tie to dashboards. ([Axios][8])
* **Deliverable.** `services/telemetry/` + Grafana JSON.
* **Value.** Makes “slop” visible; aligns with recent HBR/Stanford-BetterUp findings. ([Harvard Business Review][9])

---

## 12) A/B Harness for Brownfield Tasks (2–3 days)

* **Scope.** Randomize tasks with/without agents; log time, defects, rework; replicate RCT-style measurement for your repos. ([arXiv][1])
* **Deliverable.** `apps/ab-harness/` + guidance to design fair trials.
* **Value.** Local truth > anecdotes; prevents cargo-culting.

---

## 13) Package as **Claude Skills** (1–2 days)

* **Scope.** Wrap common flows (Research→Plan, PR-stacking, Ralph) as reusable **Skills** deployable across Claude Code, API, Agent SDK. ([The Verge][10])
* **Deliverable.** `skills/fic.*` with install docs.
* **Value.** One-click reuse; governed configuration.

---

## 14) Security/Governance (1–2 days)

* **Scope.** Repo allow/deny lists, PII/redaction, tool sandboxing; provenance in PR body; audit log of agent actions.
* **Deliverable.** `policies/security.yml`, `services/audit/`.
* **Value.** Safe productionization.

---

## 15) Pilot on a Real Brownfield (3–5 days)

* **Scope.** Pick 1 repo (≥100k LOC). Run Phases I→III on 1 bugfix + 1 small feature.
* **Deliverables.** 2 stacked PRs, `research.md`, `plan.md`, MAR page, A/B results.
* **Success.** TTFPR ↓, Merge-without-change% ↑, Rework% ↓ vs baseline.

---

## 16) Team Rollout & Playbooks (1–2 days)

* **Scope.** Lightweight SOPs: when to use **sub-agents** vs **Ralph**, compaction cadence, PR size caps, reviewer rotation.
* **Deliverables.** `docs/playbooks/*.md`; lunch-and-learn deck.
* **Value.** Cultural adoption > model choice; matches industry trend. ([Stanford HAI][11])

---

### Implementation Notes (Terse)

* **Why context-first?** Context quality dominates outcomes; naive chat histories saturate and mislead. ([Anthropic][3])
* **Stateless mental model.** Treat each step as `f(context) → action`; persist *artifacts*, not chats.
* **Sub-agents by default.** Isolate high-noise discovery; return structured summaries only. ([Claude Docs][4])
* **Specs > code review.** Review `research.md/plan.md` first to keep **mental alignment** high. ([Blake Smith][6])
* **Expect variance.** Field experiments show slowdowns in mature repos; use A/B to localize wins. ([arXiv][1])

---

## Shipping Checklist (Per Mini-Project)

* **Outputs:** code under `fic-kit/`, sample repo, docs, CI job, demo script.
* **Timebox:** each 0.5–5 days; independent; compose via CLI.
* **Done means:** runnable demo + README + metric hook.

---

### Key Sources (for the skeptics)

* **Context engineering & SDK:** Anthropic on context engineering; Agent SDK; sub-agents; Skills. ([Anthropic][3])
* **Productivity/slop evidence:** METR/Becker RCT (AI slowed pros ~19%); HBR/BetterUp/Stanford “workslop”; press coverage. ([arXiv][1])
* **Code review = mental alignment:** Blake Smith; supporting slides. ([Blake Smith][6])
* **Rework metric:** Corrective Commit Probability (CCP). ([arXiv][7])

---

**Next action** (fastest path to value): ship **1, 2, 3, 10, 11** first (≈1–2 weeks). This flips the failure modes—**clean context, small diffs, measurable slop**—before scaling to the rest.

[1]: https://arxiv.org/abs/2507.09089 "[2507.09089] Measuring the Impact of Early-2025 AI on ..."
[2]: https://www.anthropic.com/engineering/building-agents-with-the-claude-agent-sdk "Building agents with the Claude Agent SDK"
[3]: https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents "Effective context engineering for AI agents"
[4]: https://docs.claude.com/en/docs/claude-code/sub-agents "Subagents"
[5]: https://devtalk.com/t/ralph-wiggum-as-a-software-engineer/206861 "Ralph Wiggum as a \"software engineer\""
[6]: https://blakesmith.me/2015/02/09/code-review-essentials-for-software-teams.html "Code Review Essentials for Software Teams - Blake Smith"
[7]: https://arxiv.org/abs/2007.10912 "The Corrective Commit Probability Code Quality Metric"
[8]: https://www.axios.com/2025/09/24/ai-workslop-workplace-efficiency-study "AI \"workslop\" sabotages productivity, study finds"
[9]: https://hbr.org/2025/09/ai-generated-workslop-is-destroying-productivity "AI-Generated “Workslop” Is Destroying Productivity"
[10]: https://www.theverge.com/ai-artificial-intelligence/800868/anthropic-claude-skills-ai-agents "Anthropic turns to 'skills' to make Claude more useful at work"
[11]: https://hai.stanford.edu/ai-index/2025-ai-index-report "The 2025 AI Index Report | Stanford HAI"
