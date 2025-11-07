# Program Setup Tasklists & Checklists (Nov 2025)

1. **Governance & Metrics (Spec 0.1 Goals & Guardrails)**
   - **Tasks**
     - G1 Appoint metric owners + escalation path; capture in `docs/goals.md` along with purpose (why context headroom, why Claude SDK) so newcomers grok stakes.
     - G2 Define precise formulas + data sources for TTFPR, CCP/Rework%, Reviewer load, Four Keys; map Conventional Commit types + SemVer bump policy into same doc.
     - G3 Implement lightweight extractor (Python/Node script under `services/telemetry/`) hitting GitHub REST/GraphQL with PAT, caching raw JSON + derived CSV/metrics.md for last 30–90 days.
     - G4 Encode event/label conventions (Conventional Commits, issue labels for tasks, corrective detection regex/model hints) inside `policies/context.yml` for reuse by later agents (Sections 2–16 of @spec/project.md).
     - G5 Define guardrail knobs: context headroom (40–60%), small PR default (<400 LoC touched), pre-commit hygiene, revert protocol; reflect in CLI defaults + docs.
   - **Checks**
     - V1 Baseline report reproduces manual GitHub queries within ±5%; archived in repo and dashboard screenshot included.
     - V2 Guardrail policies referenced by CLI/tests (failing run if tokens >60%); manual dry-run proves enforcement.
     - V3 Metric ownership + alert thresholds reviewed with Eng leadership (sign-off recorded in docs/goals.md).

2. **Repo Skeleton & Tooling (Spec 0.2)**
   - **Tasks**
     - R1 Scaffold `fic-kit/` tree: `apps/cli`, `agents/worker`, `agents/subagent`, `skills/`, `services/{code-index,log-distill,telemetry}`, `policies/`, `docs/`, `.github/workflows/`.
     - R2 Initialize Claude Agent SDK project under `agents/worker/` with tool registry (FS, shell, git, tests) + `fic run <task>` entry wired through `apps/cli/`.
     - R3 Stub `agents/subagent/` (JSON-only contracts) + `skills/` (Research→Plan, PR stacking) to align with future phases while remaining no-op placeholders for Day 1 demo.
     - R4 Add hygiene files: MIT `LICENSE`, `README.md` (install/run), `.editorconfig`, `.pre-commit-config.yaml`, `.gitignore`, `.env.example` (GITHUB_TOKEN), `PULL_REQUEST_TEMPLATE.md`.
     - R5 Implement `fic doctor` (checks SDK install, token, pre-commit) + `fic demo` (dry-run Research→Plan using sample repo + synthetic `research.md`/`plan.md`).
   - **Checks**
     - V4 `pre-commit run --all-files`, `fic doctor`, `fic demo` all succeed from clean clone on macOS/Linux; capture logs in docs.
     - V5 Directory tree matches spec and is free of placeholder TODOs (no empty dirs in git except via `.gitkeep`).
     - V6 CLI help text documents every command and references policies.

3. **Automation & Telemetry Pipelines**
   - **Tasks**
     - A1 Author `.github/workflows/ci.yml` (PR + push) calling lint/test matrix, artifact uploads, pre-commit check, and failing if guardrail policy violated (token headroom, missing tests flag).
     - A2 Create `.github/workflows/telemetry.yml` scheduled nightly to run extractor, compute TTFPR/CCP/Reviewer load, post badge to README (shields.io or gh-pages) and commit metrics artifact to branch/tag.
     - A3 Wire `services/telemetry/` CLI (`fic metrics sync`) to run extractor locally, storing snapshots under `services/telemetry/history/` with date-stamped files for reproducibility.
     - A4 Ensure secrets management: document required GitHub PAT scopes, store Action secrets, forbid net-unsafe commands in workflows (matching policies/security.yml stub).
   - **Checks**
     - V7 Trigger CI + telemetry workflows on test branch; confirm statuses, artifacts, badges update.
     - V8 Telemetry job reuses same script as local CLI (no drift) verified via hash or shared module.
     - V9 Workflows linted via `act` or `gh workflow view` and pass shellcheck/yamllint (if available).

4. **Docs, Policies, Templates**
   - **Tasks**
     - D1 Populate `docs/goals.md` with metric rationales, formulas, baseline tables, owner table, alert thresholds.
     - D2 Write `docs/guardrails.md` covering token headroom, unsafe-change blocklist, revert protocol, pre-commit hygiene, rationale referencing long-context degradation research.
     - D3 Draft `docs/MAR.md` describing Mental-Alignment Review pack flow (review `research.md/plan.md` first, link to future MAR site from @spec/project.md §10).
     - D4 Fill `policies/diff.yml`, `policies/security.yml`, `policies/context.yml` capturing PR size caps, SemVer mapping, tool allow/deny lists, provenance requirements.
     - D5 Ensure `.github/PULL_REQUEST_TEMPLATE.md` links to research/plan artifacts, mandates tests, risks, rollback, metric impact; add issue templates if needed for consistent data.
   - **Checks**
     - V10 Docs self-contain references (no “see other doc” without link); run markdown link checker.
     - V11 Policies consumed by CLI/tests (e.g., diff cap check) proven via unit/integration tests or recorded manual run.
     - V12 PR template renders correctly on GitHub preview and enforces checkboxes for scope/tests/metrics.

5. **Acceptance, Demo, & Rollout**
   - **Tasks**
     - S1 Run end-to-end dry-run on sample repo: capture `research.md`, `plan.md`, MAR preview, metrics snapshot, and attach to docs as canonical example.
     - S2 Document install/run instructions in README (`clone`, `install deps`, `pre-commit install`, `fic doctor`, `fic demo`, telemetry setup) with troubleshooting FAQ.
     - S3 Host quickstart playbook (docs/playbooks/day0.md) summarizing when to use sub-agents vs Ralph loop, aligning with @spec/project.md next steps.
     - S4 Present success criteria checklist (CI green, telemetry baseline captured, guardrails enforced) and secure stakeholder sign-off (record date + approver in docs/goals.md).
   - **Checks**
     - V13 Demo log + screen capture stored under `docs/demos/` and referenced from README; reproducibility verified by second engineer.
     - V14 Success criteria table ticks all boxes (clone/run, CI green, telemetry baseline, Skills dry-run) with timestamps (Nov 2025) to prevent ambiguity.
     - V15 Stakeholder approval + follow-up tasks for phases 1–3 captured as GitHub issues linked from docs.
