---
name: yf-eng-plan
description: Plans software product engineering work in two phases — a product spec (product-spec.md, with wireframes where UI applies), then after explicit sign-off a technical spec (technical-spec.md) with a phased implementation contract for /yf-eng-build — using research subagents and adversarial review. Use when the user invokes /yf-eng-plan or asks to plan a software feature, product, or system change, or wants a product spec, technical spec, or engineering plan. For non-software planning use a general plan skill. To execute a signed-off technical spec, use yf-eng-build.
---

# Engineering planning (yf-eng-plan)

Two-phase planning for software product engineering: a **product spec** (what and why), then — only after explicit user sign-off — a **technical spec** (how) that `/yf-eng-build` can execute.

## Models (Task allowlist — exact Cursor slugs)

| Work | `model` | `subagent_type` |
|------|---------|-----------------|
| Research / design assets | `cursor-grok-4.5-high` | `explore` or `generalPurpose` |
| Adversarial review | `gpt-5.6-sol-medium` | `generalPurpose` |

Pass `model` and `subagent_type` on every Task call. Unsupported slugs are a hard preflight error — do not silently substitute.

## Orchestration rules

- You (the base agent) are the orchestrator and own the final result. Subagents inform; you decide.
- Every Task prompt must be self-contained: subagents have no conversation history, so include the user's goal, all relevant context, and exactly what to return.
- Run independent research units in parallel (multiple Task calls in one message).
- Every research finding must cite the file path(s) supporting it. No path, no claim.
- Reviewers receive document **paths only** — never paste document bodies into prompts. Reviewers must not trust anything the document claims; they verify independently and **edit the file directly**.
- Accept reviewer edits unless one clearly contradicts an explicit user answer (from clarifying questions or a sign-off); only then revert that edit.

Ground the technical spec in the **actual codebase** (and its `docs/adr/`, `docs/architecture/` when present). Phasing and validation must be discoverable from that research — do not invent an implementation sequence that ignores existing structure.

## Writing for human review

Each spec has two readers: the **user**, who decides whether to sign off, and the **build agent**, which executes the contract. The document body serves the user; machine-level detail goes in the Appendix. Rules for the body:

1. Open with a **Read this first** section (half a page max): the problem in one sentence, the chosen approach in plain language, the 3–5 decisions that need the user's judgment, and what is risky. The user should be able to sign off from this section alone.
2. Define every specialized term on first use; add a short **Glossary** subsection when more than a few are unavoidable.
3. Keep paragraphs to ~3 sentences. Prefer short declarative sentences over dense prose.
4. Frame each architecture decision as **Decision / Why / What we rejected** — not analysis prose.
5. In the technical spec, keep the Implementation sequence body to a short plain-English entry per phase; the full per-phase contract (scope, acceptance, validation commands, done-when) lives in the **Appendix**, referenced by phase ID.

## Phase 1 — Product spec

**Research and clarifying questions**

1. Decompose the user's prompt into independent search/synthesis units. For each unit, launch a Task subagent (`subagent_type: explore` or `generalPurpose`) with `model: cursor-grok-4.5-high`. Each prompt must state the user's goal, the specific question, where to look, and a return format of path-cited bullets.
2. Ask the user at most **5** questions whose answers will **materially shape the plan** (architecture shape, scope boundaries, stakeholders, hard constraints, success criteria). Skip nice-to-know; if answers open a new fork, ask a focused follow-up round.

**Draft:** write the spec to disk as `product-spec.md`, or `<slug>-product-spec.md` if the location will hold multiple specs. Colocate it with the relevant project (often the company wiki / project folder — not the code repo’s `docs/`). If the right location is unclear, ask. Template:

```markdown
# <Product title> — Product Spec

## Read this first
## Problem / opportunity
## Goals & non-goals
## Users & stakeholders
## Success criteria
## Scope
### In scope
### Out of scope
### Later
## Experience
### Journeys & key flows
### Design drafts / wireframes
## Requirements
### Must
### Should
### Constraints
## Assumptions & open questions
## Risks & mitigations
## Dependencies
```

For non-UI work (backend services, pipelines, infrastructure), mark **Experience** as `N/A — <why>` and skip wireframes.

**Design drafts (where UI applies):** include rough wireframes in **Design drafts / wireframes**, produced via `cursor-grok-4.5-high` Task subagents. ASCII or mermaid embedded in the markdown is fine — capture visual hierarchy and key states (empty, loading, error, populated), not pixel detail.

**Adversarial review:** launch one `gpt-5.6-sol-medium` `generalPurpose` reviewer with the product-spec **path only**. The reviewer verifies against the codebase and docs via its own `cursor-grok-4.5-high` subagents, also enforces the **Writing for human review** rules, and edits the file directly; apply the acceptance rule above.

**Sign-off gate:** present the reviewed product-spec and **wait for explicit sign-off**. Do not begin Phase 2 without it. If the user requests changes, revise (re-running review if the changes are substantial) and present again.

## Phase 2 — Technical spec (only after sign-off)

**Draft:** you (the base agent) draft the plan with the product-spec in context — Read the product-spec file directly. Write it as `technical-spec.md`, or `<slug>-technical-spec.md` alongside the matching product-spec. Template:

```markdown
# <Title> — Technical Spec

## Read this first
## Context & touchpoints
## Architecture & key decisions
## Interfaces & contracts
## Data model & migrations
## Implementation sequence

### Phase: <id-slug>
- **Intent:** one plain-English sentence — the observable state that becomes true
- **Depends on:** phase ids or `none`
- **Contract:** Appendix — <id-slug>

## Testing strategy
## Rollout, flags & rollback
## Observability & security
## Documentation plan
## Open technical risks
## References
## Appendix — implementation contract

### <id-slug>
- **Scope:** in / explicit out
- **Acceptance:** criteria traceable to the product spec
- **Test strategy:** one thin acceptance check — automated / characterization / build-typecheck / other (justify)
- **Validation:** exact commands or procedures, env prerequisites, destructive/external notes
- **Done when:** checklist of observables
- **Docs expected:** none | ADR(s) | architecture concern update | AGENTS.md (as applicable)
```

### Implementation sequence (required contract for `/yf-eng-build`)

Every phase appears twice: a body entry (Intent, Depends on) the user can read as a narrative, and an Appendix entry carrying the full contract fields in the template above. Dependency order must be explicit. Validation commands must be runnable (or explicitly blocked on missing env). If research cannot fill a field, leave it as an open question — do not paper over gaps.

**Testing strategy (default):** per-phase testing is thin — one smoke-level acceptance check that proves the phase's Intent, not edge-case breadth. Comprehensive testing (edge cases, regression matrix, full test suite) is a dedicated **`test-hardening`** phase, last in the implementation sequence: it depends on all build phases and is gated on the user confirming the implementation by manual use of the product. Include it in every implementation sequence unless the user specifies otherwise; its Appendix entry states what the comprehensive suite must cover, or that a test-plan document is the deliverable instead.

**Documentation plan:** state which ADRs / architecture concern files / AGENTS.md updates `/yf-eng-build` should produce when decisions land (following the target repo’s layout, e.g. `docs/adr/`, `docs/architecture/concerns/`). Binding decisions belong in the code repo as ADRs at build time — not only in this plan. Product/strategy stays in the wiki.

**References** must link the product-spec path and every source file the plan relies on (including existing ADRs/architecture docs consulted).

**Adversarial review:** launch a `gpt-5.6-sol-medium` review subagent with one delta — pass the **paths to both** the product-spec and the technical-spec (paths only, no pasted bodies). The reviewer trusts neither document, verifies against the actual code and docs via its own `cursor-grok-4.5-high` subagents, and **edits only the technical-spec file** — the product-spec is signed off and frozen. Apply the acceptance rule, treating the signed-off spec as an explicit user decision. Reviewer must attack weak phases (missing Validation, vague Acceptance, ungrounded Docs expected) **and unreadable writing** (undefined jargon, dense paragraphs, decisions buried in prose, implementation detail leaking out of the Appendix into the body).

**Deliver:** present the final technical-spec, summarizing key architecture decisions, the phased implementation sequence, docs/ADR expectations, and open technical risks needing the user's input. Note that execution is `/yf-eng-build`.

## Guardrails

- Never skip the product-spec phase, even for "obviously technical" work — a minimal spec is still required unless the user explicitly waives it.
- Never start Phase 2 before explicit user sign-off on the product-spec.
- Both documents live on disk, not only in chat.
- Do not put master plans or product-strategy docs into the code repo; keep them in the wiki / project folder.
