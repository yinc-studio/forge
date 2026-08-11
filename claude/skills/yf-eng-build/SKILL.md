---
name: yf-eng-build
description: Executes a signed-off technical-spec.md in the target codebase with phased test-first implementation, orchestrator-observed validation, adversarial review, in-repo build-run records, and ADR/architecture doc updates. Use when the user invokes /yf-eng-build or /forge:yf-eng-build or asks to implement, build, or execute a technical spec or yf-eng-plan output. Not for product/tech planning — that is yf-eng-plan.
---

# Eng-build (execute technical spec)

Implement a signed-off `technical-spec.md` in the **codebase repo** where the changes land. Status is `complete`, `pending-manual-validation`, `partial`, or `blocked` — never “done” on vibes.

Per-phase testing is deliberately thin — one acceptance check proving the phase's Intent — to shorten the path to the user manually using the product. Comprehensive testing (edge cases, regression matrix, full suite) is the trailing `test-hardening` phase, which runs after the user confirms the implementation by manual use (unless the technical spec or the user specifies otherwise).

## Models (exact Claude model IDs)

| Work | `model` | Agent |
|------|---------|-------|
| Read-only context | `claude-sonnet-5` | `Explore` |
| Straightforward builder | `claude-sonnet-5` | `general-purpose` |
| Complex / multi-file builder | `claude-opus-5` | `general-purpose` |
| Adversarial code review | `claude-fable-5` | `general-purpose` |

Pass `model` explicitly on every Task/Agent call using the exact ID from the table above, and set reasoning effort to `high`. Unsupported IDs are a hard preflight error — do not silently substitute. When `/yf-eng-build` is active, this routing overrides `yf-orchestrate`.

## Invariants

1. Base agent is orchestrator; only it may declare final status.
2. **Single writer:** at most one actor edits the working tree at a time (builder, then reviewer).
3. No parallel phase execution in v1. Parallel **read-only** context only.
4. Workers do not spawn nested workers; orchestrator routes everything.
5. Path-cited claims only. Path-only review handoffs (no pasted doc bodies).
6. Orchestrator **observes** validation (shell exit codes). Agent claims are not evidence.
7. Product-spec and technical-spec are **read-only** during `/yf-eng-build`. Reviewer edits code/tests/docs artifacts only — never the signed-off plans or evidence logs.
8. Accept reviewer edits unless they clearly contradict the signed-off docs or an explicit user answer; then stop and surface the conflict.
9. Do not commit/push/PR/deploy unless the user separately authorizes it.

## Inputs

- **Required:** path to signed-off `technical-spec.md` (ask to confirm sign-off if not recorded).
- Read linked `product-spec.md` from References.
- Optional: phase ID to limit/resume.

### Weak-plan gate

At intake, classify gaps:

1. **Mechanical** — unambiguous from repo scripts/paths; normalize into the build-run record only (do not edit signed-off docs).
2. **Semantic** — acceptance, architecture, scope, deps, migration/safety unclear → stop; ≤5 questions or send back to `yf-eng-plan`.
3. **Contradiction / infeasible** — stop with path-cited evidence; do not invent a new design.

## Build-run record (in the codebase repo)

Create/update a durable run record **in the repo being changed** (not only in chat):

- Default: `docs/builds/YYYY-MM-DD-<slug>.md` when `docs/` exists; otherwise `builds/YYYY-MM-DD-<slug>.md`.
- If the repo documents a different builds location, follow that.
- Include: technical-spec + product-spec paths, phase statuses, acceptance→evidence map, changed paths, validation commands + exit codes, RED evidence, review summary, blockers, doc/ADR paths written.
- Resume only via phase ID after re-reading both signed-off docs, this record, and the current diff. If the record is missing/stale, reconstruct from paths + repo state — do not trust chat summaries.

## Documentation (same change set)

Treat docs as part of the build, not a follow-up. Mirror the yinc-platform layout when present (`docs/adr/`, `docs/architecture/`, `CONTRIBUTING.md`); otherwise follow the repo’s equivalent or ask once.

**During / after relevant phases (before calling the work complete):**

1. **ADRs** — for fork-in-the-road decisions (boundaries, invariants, rejected alternatives, non-obvious schema/API policy). Use `docs/adr/_template.md` when it exists. One decision per ADR. Update `docs/adr/README.md` (or index) when adding.
2. **Architecture** — structural changes update `docs/architecture/overview.md` and/or the relevant `docs/architecture/concerns/*.md` in the same change set. Prefer concern files over bloating overview.
3. **AGENTS.md** — update colocated/root agent runbooks when boundaries, commands, or validation loops change.
4. **Wiki link-back** — if a wiki technical-spec/product-spec exists, add a **Promoted decisions** link to new ADRs; do **not** paste ADR bodies into the wiki. Do not put product/strategy docs into the codebase.
5. **Sensitivity** — keep business/GTM/client/pricing material out of the code repo when the repo’s contributing rules forbid it (e.g. Tier B).

Do **not** write binding decisions only in chat or only in the wiki when the repo uses ADRs — promote into `docs/adr/` as part of `/yf-eng-build`.

## Execution flow

### 1. Intake and baseline

- Verify both signed-off docs on disk; apply weak-plan gate.
- Discover docs layout (`docs/adr`, `docs/architecture`, CONTRIBUTING/AGENTS).
- Preserve unrelated working-tree changes.
- Resolve validation commands, env, cost, side effects; ask before destructive/external/paid/production checks.
- Run a **safe baseline** of planned validation; record pre-existing failures. Broken baseline must be fixed, explicitly scoped out, or block — not ignored by comparing only finals.
- Create the build-run record.

### 2. Per phase (dependency order)

1. Orchestrator prepares a path-cited phase brief; pick builder model by complexity.
2. **Builder** (single writer for the phase):
   - Add or identify **one thin acceptance check** (test or evidence procedure) for **this phase** — enough to prove the phase's Intent happened, not edge-case breadth. Edge cases, regression matrices, and comprehensive suites belong to the `test-hardening` phase.
   - **Mandated failing run:** when a suitable automated harness exists for new behavior or a bug fix, orchestrator must observe a **RED** run of that check that fails for the intended reason *before* implementation. Exceptions (record in the build-run): existing coverage already asserts the criterion; characterization-only legacy; config/docs/generated/UI/infra where the phase’s Validation is non-test evidence — still require the plan’s stated evidence procedure. A RED caused by syntax/env/unrelated failure does **not** count.
   - Implement the phase scope.
3. Orchestrator runs phase Validation + required regressions; must be green.
4. On failure: at most **two** evidence-based repair loops; then `partial`/`blocked` with logs. No retry without a new hypothesis.
5. Launch **one** reviewer (`claude-fable-5`) with paths only: product-spec, technical-spec, phase ID, repo root, changed-files manifest path, validation/build-run paths. Reviewer distrusts the docs, checks the repo, may fix code/tests/docs; must not weaken tests to pass; must flag plan contradictions instead of redesigning.
6. Re-validate after any reviewer edit.
7. Mark phase complete only when the phase gate passes; else do not start dependents.

### 3. Manual validation, then test hardening

- After all implementation phases pass their gates, present the build to the user for **manual validation** — using the product, not reading logs. Set status `pending-manual-validation` in the build-run record. This can span sessions; resume via the `test-hardening` phase ID.
- Once the user confirms the implementation by manual use, execute the **`test-hardening`** phase from the technical spec: edge cases, regression matrix, and the comprehensive test suite — or, when the spec says so, a test-plan document specifying that suite. Skip only when the technical spec or the user explicitly says so.
- If the spec predates this contract and has no `test-hardening` phase, derive one from its Testing strategy section and record it in the build-run (mechanical gap).
- Test hardening follows the same per-phase mechanics (builder → orchestrator-observed validation → reviewer), minus the RED mandate — its tests assert behavior that already exists and must pass green.

### 4. Final

- Run the authoritative full safe suite from plan + repo instructions (not merely the union of targeted tests). Report exactly what did/didn’t run.
- Finish required ADR/architecture/AGENTS updates; link from build-run + wiki Promoted decisions as applicable.
- Audit diff vs phase scope and acceptance→evidence map.
- Status `complete` only if the final completion gate passes; else `partial` or `blocked`.

## Gates

### Phase complete iff

- Dependencies complete.
- Every acceptance criterion maps to evidence (path or logged procedure).
- Mandated RED was observed when required, then Validation green under orchestrator observation.
- No required check skipped (waiver → phase stays `partial`, risk disclosed).
- Review findings fixed, user-accepted as residual risk, or proven inapplicable with paths.
- Reviewer edits revalidated.
- No unexplained scope creep, disabled/weakened tests, or accidental unrelated edits.
- No unresolved plan contradiction.

### Build complete iff

- Every requested phase passed its phase gate.
- User confirmed the implementation by manual use, and the `test-hardening` phase passed its gate (or was explicitly waived by the spec or the user). Until then the ceiling is `pending-manual-validation`.
- Full safe suite passed after the last edit.
- Required docs/ADR/architecture/AGENTS updates for the change are done (or explicitly N/A with reason in the build-run).
- Report distinguishes new vs pre-existing failures, waivers, and checks that could not run.

**Forbidden:** deleting/skipping tests, weakening assertions, changing validation commands, regenerating snapshots without inspection, or narrowing scope solely to go green.

## Default roles

| Role | Who |
|------|-----|
| Orchestrator | Base agent — sequencing, shells, evidence, acceptance, docs checklist, final status |
| Builder | One Task/Agent per phase — tests/evidence then implementation |
| Reviewer | One Task/Agent per phase after green validation — independent fix/report |

Do not default to a separate test-author agent. Split test design from implementation only for unusually high-risk acceptance tests, and then only **sequentially** with disjoint ownership stated in both prompts.
