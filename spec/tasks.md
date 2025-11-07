# FIC Kit Implementation + Verification Tasklists (Nov 2025)
Scope: deliver spec/spec.md for a measurable, acquirable brownfield FIC repo + Claude Worker; assume zero prior context; optimize for 80/20 impact.

1. Program Foundations (§0.1)
   1.1 Tasks
       - Baseline metrics: pull last 30–90d Issues/PRs to compute TTFPR, CCP, reviewer load, Four Keys; log formulas + query links in docs/goals.md and snapshots folder.
       - Codify conventions: document Conventional Commits→SemVer mapping, corrective commit regex, label taxonomy for tasks/PRs, and reviewer WIP caps.
       - Guardrails: author docs/guardrails.md covering token headroom policy, unsafe-change blocklist, revert flow, and reviewer expectations.
       - Policy gates: add lightweight rules (context headroom enforcement, small-PR default, pre-commit hygiene) into policies/context.yml and diff/security policies.
       - Data sourcing: define REST/GraphQL collectors, auth scopes, snapshot cadence, and storage format (JSON/Parquet) for dashboards.
   1.2 Checks
       - Each metric shows formula, baseline value, data source, and SLA in docs/goals.md with timestamps.
       - Guardrails doc references Nov 2025 headroom caps (≤60% tokens) and rollback SOP; reviewers confirm readability.
       - Policy files enforce pre-commit+PR template gating via automated test run.
       - Daily snapshot job proves two consecutive runs stored with hash-stamped files.

2. Repo Skeleton & Hygiene (§0.2)
   2.1 Tasks
       - Create top-level layout: apps/cli, agents/worker+subagent, skills/, services/code-index|log-distill|telemetry, policies/, docs/, .github/workflows/.
       - Seed docs/goals.md, docs/guardrails.md, docs/MAR.md with references back to metrics + guardrails.
       - Add hygiene files: .editorconfig (LF, UTF-8, per-language indentation), .pre-commit-config.yaml (formatters, linters, secrets scan), LICENSE (MIT), README, PR template.
       - Define policies diff.yml/security.yml/context.yml capturing gating logic and risk callouts.
       - Implement workflows: ci.yml (PR+push matrix, artifact logs) and telemetry.yml (nightly metrics, badge update).
   2.2 Checks
       - `rg --files` lists every required path; README links to each artifact.
       - `pre-commit run --all-files` is clean on fresh clone.
       - CI workflow passes on sample PR; telemetry workflow logs scheduled timestamp and badge artifact.
       - PR template renders scope/tests/risk/rollback checklist via GitHub preview.

3. Install & Conventions (§0.2 Install/Run + Conventions)
   3.1 Tasks
       - Document one-screen install: clone, package manager install (pipx/uv/poetry or pnpm), pre-commit install, GITHUB_TOKEN export, `fic doctor`, `fic demo`.
       - Build `fic` CLI entry wrapping agent runs, telemetry, and doctor flows; ensure discoverable help text.
       - Define secrets handling (env vars, CI secrets) and forbid unsafe CLI flags unless approved.
       - Encode commit/tag policy: Conventional Commits drive SemVer bumps + CCP tagging, with automation notes.
   3.2 Checks
       - Fresh workstation walkthrough yields zero manual fixes; `fic doctor` validates SDK, Skills, tokens.
       - `fic demo` completes Research→Plan dry-run with stored sample repo outputs.
       - README section shows conventions table; sample commit history proves autop SemVer bump.

4. Metrics Automation & Dashboards (§0.3)
   4.1 Tasks
       - Implement scripts/services to compute TTFPR, CCP, reviewer load, Four Keys via GitHub REST/GraphQL with retry/backoff.
       - Store results as dated artifacts (JSON/CSV) in telemetry/ with links surfaced in docs/goals.md.
       - Wire telemetry.yml to run nightly with caching + badge publication (shields-compatible endpoint).
       - Provide lightweight dashboard (mdx/html) summarizing trends + deltas vs baseline.
   4.2 Checks
       - Local dry-run prints sample metrics and matches manual calculations.
       - Nightly workflow run retains artifacts for ≥7 days and updates badge URL.
       - Dashboard renders without JS build steps and references latest snapshot timestamp.

5. Day-1 Success Readiness (§0.4)
   5.1 Tasks
       - Capture acceptance checklist proving repo clones, CI green, pre-commit active, PR template enforced.
       - Ensure telemetry dashboard shows baseline for all primary metrics before launch.
       - Validate Skills + Agent SDK via scripted `fic demo` dry-run and log outputs.
   5.2 Checks
       - Checklist stored at docs/MAR.md with signed owner + date.
       - `fic demo` log attached to MAR demonstrating Claude Agent SDK success.
       - Baseline metrics screenshot/badge archived for audit.

6. Worker Implementation (§SDK §0–4)
   6.1 Tasks
       - Provision runtimes (Node18+/Py3.10+), install Claude Agent SDK + Claude Code CLI, and document ANTHROPIC_API_KEY requirements.
       - Implement worker (choose TS or Py) exactly as minimal loop: load CLAUDE.md via settingSources:['project'], preset claude_code system, allowed tools set, MCP server for custom tools.
       - Add CLI command `fic worker` (and config) launching worker with toggles for plan/dry-run/acceptEdits/bypass.
       - Ensure worker streams outputs, logs tool uses compactly, and respects cwd isolation.
   6.2 Checks
       - Running `fic worker --plan` shows streaming tokens with no edits; switching to acceptEdits applies file changes.
       - Worker logs confirm allowedTools == {Read,Grep,Glob,Edit,Write,Bash,WebFetch,WebSearch} unless overrides supplied.
       - CLAUDE.md instructions visibly loaded (test via instrumentation echo).

7. Permissions & Guardrails (§5)
   7.1 Tasks
       - Configure default permission ladder (start default, escalate to acceptEdits, forbid bypass outside sandbox) with CLI flags + policy docs.
       - Implement `canUseTool` plus Pre/PostToolUse hooks enforcing context headroom, bash command allow-lists, diff lint/test runs after edit write.
       - Encode declarative rules in .claude/settings.json (path jail, domain restrictions, risk prompts).
       - Build auto-compaction + headroom rejection handling so oversized contexts are compacted before retry.
   7.2 Checks
       - Attempted forbidden Bash (e.g., `rm -rf /`) blocked with logged reason.
       - PostToolUse proves formatter/test hook execution after Edit; failure aborts session.
       - Permission switch attempts recorded; bypass mode requires explicit confirmation flow.

8. Tools, Subagents, Custom MCP (§4,6,7)
   8.1 Tasks
       - Wire first-class tools: text editor tool (20250728), Bash, code execution, WebFetch, WebSearch (with domain allow-lists + max_uses).
       - Define subagents (research, planner, implementor, QA) each with prompts, allowed tools, models, and context isolation strategy; expose via options.agents.
       - Implement reusable Claude Skills bundles under skills/ (Research→Plan, PR stacking) and reference from Worker.
       - Build custom MCP tools (e.g., echo template plus project-specific services) via SDK servers and register in worker options.
   8.2 Checks
       - Subagent orchestration test: research→planner→implementor pipeline completes with isolated contexts noted in logs.
       - Skills referenced by name load successfully; worker fails fast if missing.
       - WebSearch honoring domain allow-list (intentionally blocked domain recorded) and citations captured.

9. Hosting, Security, Sandboxing (§8–9)
   9.1 Tasks
       - Containerize worker with minimal Node/Python + Claude Code CLI; define ENTRYPOINT for `fic worker` and health checks.
       - Specify sandbox policy: per-task ephemeral container, resource limits, network egress allowlist (api.anthropic.com, GitHub APIs), filesystem jail.
       - Document secret management (env vars, CI secrets) and ensure telemetry + agents use repo-level secrets.
       - Provide guidance for deployment modes (ephemeral, long-running, hybrid, single-container) with when-to-use notes.
   9.2 Checks
       - `docker build` succeeds; container runs `fic worker` smoke test with mocked API key.
       - Sandbox denies disallowed network targets and file writes outside workspace.
       - Secrets scan ensures no plaintext keys committed; CI proves env injection works.

10. Operability & DX (§10–11)
    10.1 Tasks
        - Implement resume/fork session controls, `maxTurns`, and automatic compaction boundaries; surface via CLI flags.
        - Handle streamed SDKMessage union types (assistant/user/result/system/partial/compactBoundary) with structured logging.
        - Create error-handling layer translating CLI/JSON/tool failures into actionable user prompts.
        - Document runtime switches (permissionMode commands, allowed tool narrowing) for operators.
    10.2 Checks
        - Simulated crash resumes session from persisted state.
        - Logs show partial + compactBoundary markers without garbling.
        - Operator guide validated by dry-run toggling allowedTools mid-session.

11. Runtime QA & Smoke Tests (§12)
    11.1 Tasks
        - Script plan-only test ensuring no edits/tools requiring write run before approval.
        - Script edit flow test: Worker edits dummy file, formatter + lint/test hooks fire, diffs shrink post-format.
        - Script web flow test: single search query with allow-list + citation check.
        - Store results + expected outputs for regression comparison.
    11.2 Checks
        - Plan-only run produces plan artifact and zero file diffs; log stored.
        - Edit flow emits formatter/test logs and stops on failure; success case commits formatted diff.
        - Web flow outputs citation metadata; blocked domain attempt recorded.

12. Rationale & Knowledge Surfacing (§0.5 + Quick Links)
    12.1 Tasks
        - Summarize rationale bullets (context headroom, Conventional Commits/SemVer, Pre-commit/EditorConfig, Four Keys+CCP) inside README and docs/goals.md for onboarding.
        - Embed quick-link references (Anthropic SDK, Lost in the Middle, DORA, CCP, PR template, pre-commit, EditorConfig) wherever the related tasklists live.
    12.2 Checks
        - README rationales align with spec text and cite source links.
        - Quick-link anchors tested to ensure every referenced doc resolves.
