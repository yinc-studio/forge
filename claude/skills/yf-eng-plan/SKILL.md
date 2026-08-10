---
name: yf-eng-plan
description: Plans software product engineering work in two phases — a product spec (product-spec.md, with wireframes where UI applies), then after explicit sign-off a technical spec (technical-spec.md) with a phased implementation contract for /yf-eng-build — using research subagents and adversarial review. Use when the user invokes /yf-eng-plan or /forge:yf-eng-plan or asks to plan a software feature, product, or system change, or wants a product spec, technical spec, or engineering plan. For non-software planning use a general plan skill. To execute a signed-off technical spec, use yf-eng-build.
---

# Engineering planning (yf-eng-plan)

Two-phase planning for software product engineering: a **product spec** (what and why), then — only after explicit user sign-off — a **technical spec** (how) that `/yf-eng-build` can execute.

## Models (exact Claude model IDs)

| Work | `model` | Agent |
|------|---------|-------|
| Research / design assets | `claude-sonnet-5` | `Explore` or `general-purpose` |
| Adversarial review | `claude-fable-5` | `general-purpose` |

Pass `model` explicitly on every Task/Agent call using the exact ID from the table above, and set reasoning effort to `high`. Unsupported IDs are a hard preflight error — do not silently substitute.

## Orchestration rules

- You (the base agent) are the orchestrator and own the final result. Subagents inform; you decide.
- Every Task/Agent prompt must be self-contained: subagents have no conversation history, so include the user's goal, all relevant context, and exactly what to return.
- Run independent research units in parallel (multiple Task/Agent calls in one message).
- Every research finding must cite the file path(s) supporting it. No path, no claim.
- Reviewers receive document **paths only** — never paste document bodies into prompts. Reviewers must not trust anything the document claims; they verify independently and **edit the file directly**.
- Accept reviewer edits unless one clearly contradicts an explicit user answer (from clarifying questions or a sign-off); only then revert that edit.

Ground the technical spec in the **actual codebase** (and its `docs/adr/`, `docs/architecture/` when present). Phasing and validation must be discoverable from that research — do not invent an implementation sequence that ignores existing structure.

## Phase 1 — Product spec

**Research and clarifying questions**

1. Decompose the user's prompt into independent search/synthesis units. For each unit, launch a Task/Agent subagent (`Explore` or `general-purpose`) with `model: claude-sonnet-5`. Each prompt must state the user's goal, the specific question, where to look, and a return format of path-cited bullets.
2. Ask the user at most **5** questions whose answers will **materially shape the plan** (architecture shape, scope boundaries, stakeholders, hard constraints, success criteria). Skip nice-to-know; if answers open a new fork, ask a focused follow-up round.

**Draft:** write the spec to disk as `product-spec.md`, or `<slug>-product-spec.md` if the location will hold multiple specs. Colocate it with the relevant project (often the company wiki / project folder — not the code repo’s `docs/`). If the right location is unclear, ask. Template:

```markdown
# <Product title> — Product Spec

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

**Design drafts (where UI applies):** include rough wireframes in **Design drafts / wireframes**, produced via `claude-sonnet-5` Task/Agent subagents. ASCII or mermaid embedded in the markdown is fine — capture visual hierarchy and key states (empty, loading, error, populated), not pixel detail.

**Adversarial review:** launch one `claude-fable-5` `general-purpose` reviewer with the product-spec **path only**. The reviewer verifies against the codebase and docs via its own `claude-sonnet-5` subagents, edits the file directly; apply the acceptance rule above.

**Sign-off gate:** present the reviewed product-spec and **wait for explicit sign-off**. Do not begin Phase 2 without it. If the user requests changes, revise (re-running review if the changes are substantial) and present again.

## Phase 2 — Technical spec (only after sign-off)

**Draft:** you (the base agent) draft the plan with the product-spec in context — Read the product-spec file directly. Write it as `technical-spec.md`, or `<slug>-technical-spec.md` alongside the matching product-spec. Template:

```markdown
# <Title> — Technical Spec

## Context & touchpoints
## Architecture & key decisions
## Interfaces & contracts
## Data model & migrations
## Implementation sequence

### Phase: <id-slug>
- **Intent:** observable state that becomes true
- **Scope:** in / explicit out
- **Depends on:** phase ids or `none`
- **Acceptance:** criteria traceable to the product spec
- **Test strategy:** automated / characterization / build-typecheck / other (justify)
- **Validation:** exact commands or procedures, env prerequisites, destructive/external notes
- **Done when:** checklist of observables
- **Docs expected:** none | ADR(s) | architecture concern update | AGENTS.md (as applicable)

## Testing strategy
## Rollout, flags & rollback
## Observability & security
## Documentation plan
## Open technical risks
## References
```

### Implementation sequence (required contract for `/yf-eng-build`)

Every phase must include the fields in the template above. Dependency order must be explicit. Validation commands must be runnable (or explicitly blocked on missing env). If research cannot fill a field, leave it as an open question — do not paper over gaps.

**Documentation plan:** state which ADRs / architecture concern files / AGENTS.md updates `/yf-eng-build` should produce when decisions land (following the target repo’s layout, e.g. `docs/adr/`, `docs/architecture/concerns/`). Binding decisions belong in the code repo as ADRs at build time — not only in this plan. Product/strategy stays in the wiki.

**References** must link the product-spec path and every source file the plan relies on (including existing ADRs/architecture docs consulted).

**Adversarial review:** launch a `claude-fable-5` review subagent with one delta — pass the **paths to both** the product-spec and the technical-spec (paths only, no pasted bodies). The reviewer trusts neither document, verifies against the actual code and docs via its own `claude-sonnet-5` subagents, and **edits only the technical-spec file** — the product-spec is signed off and frozen. Apply the acceptance rule, treating the signed-off spec as an explicit user decision. Reviewer must attack weak phases (missing Validation, vague Acceptance, ungrounded Docs expected).

**Deliver:** present the final technical-spec, summarizing key architecture decisions, the phased implementation sequence, docs/ADR expectations, and open technical risks needing the user's input. Note that execution is `/yf-eng-build`.

## Guardrails

- Never skip the product-spec phase, even for "obviously technical" work — a minimal spec is still required unless the user explicitly waives it.
- Never start Phase 2 before explicit user sign-off on the product-spec.
- Both documents live on disk, not only in chat.
- Do not put master plans or product-strategy docs into the code repo; keep them in the wiki / project folder.
