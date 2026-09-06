---
name: yf-eng-plan
description: Plans software product engineering work as one wiki spec with two phases — product (what/why, including decisions) then after explicit sign-off technical (how) with a phased implementation contract for /yf-eng-build. Specs live in the wiki. Outstanding work defaults to the spec Status line; Linear only when the project's PROJECT.md Data Sources says so. Scales product-phase depth to the work; always runs light research subagents. Use when the user invokes /yf-eng-plan, /forge:yf-eng-plan, or asks to plan a software feature, product, or system change. To execute a signed-off spec, use yf-eng-build.
---

# Engineering planning (yf-eng-plan)

Two-phase planning into a **single wiki spec**: **product** (what and why — including decisions and rationale), then — only after explicit user sign-off — **technical** (how) that `/yf-eng-build` can execute.

The document is the record. Chat is not.

## Models (exact Claude model IDs)

| Work | `model` | Agent |
| ---- | ------- | ----- |
| Research (code, specs, ADRs) | `claude-opus-5` | `Explore` |
| Design / wireframes | `claude-opus-5` | `Explore` or `general-purpose` |
| Adversarial review | `claude-fable-5` | `general-purpose` |
| Reviewer's nested verification | `claude-sonnet-5` | `Explore` |

Pass `model` explicitly on every Task/Agent call using the exact ID from the table above, and set reasoning effort to `high`. Unsupported IDs are a hard preflight error — do not silently substitute.

## Orchestration rules

- You (the base agent) are the orchestrator and own the final result. Subagents inform; you decide.
- Every Task/Agent prompt must be self-contained: subagents have no conversation history, so include the user's goal, all relevant context, and exactly what to return.
- Run independent research units in parallel (multiple Task/Agent calls in one message).
- Every research finding must cite the file path(s) supporting it. No path, no claim.
- Reviewers receive document **paths only** — never paste document bodies into prompts. Reviewers must not trust anything the document claims; they verify independently and **edit the file directly**.
- Accept reviewer edits unless one clearly contradicts an explicit user answer (from clarifying questions or a sign-off); only then revert that edit.

Ground the technical half in the **actual codebase** (and its `docs/adr/`, `docs/architecture/` when present). Phasing and validation must be discoverable from that research — do not invent an implementation sequence that ignores existing structure.

## Writing for human review

The spec has two readers: the **user**, who signs off, and **`/yf-eng-build`**, which executes the contract.

1. Open **Product** with a **Read this first** subsection (half a page max): problem in one sentence, chosen approach, the 3–5 decisions needing judgment, and what is risky. The user should be able to sign off on Product from this alone.
2. Define specialized terms on first use; add a short **Glossary** when more than a few are unavoidable.
3. Keep paragraphs to ~3 sentences. Prefer short declarative sentences.
4. Frame decisions as **Decision / Why / What we rejected** in **Decisions** — not buried in analysis prose.
5. In **Technical**, keep **Implementation sequence** readable: one plain-English **Intent** line per phase in the body; full per-phase contract fields (Scope, Acceptance, Validation, Done when) inline under each phase or in an **Appendix — implementation contract** when the sequence is long — never duplicate the same phase twice.

## Specs vs outstanding work — `PROJECT.md` → Data Sources

Two concerns. Do not collapse them.

| Concern | What it owns | Default |
| ------- | ------------ | ------- |
| **Engineering specs** | The combined product+technical spec | Wiki (`Location` + `Filename`) |
| **Outstanding work** | Whether the work is still to do | **Wiki** — the spec **Status** line. External tracker only if the table says so. |

Read **Data Sources** on the nearest `PROJECT.md` (project folder, then walk ID → category → area):

1. **Engineering specs** — required. `Destination` is `wiki` (needs `Location` + `Filename`). If missing or incomplete, ask — do not invent a path.
2. **Outstanding work** — optional. Omitted or `Destination: wiki` → track via spec Status only. `Destination: linear` needs `Form: issue` and `Team` (and `Project` if named). Unknown destination: ask once.

**Wiki spec:** write `spec.md` (or `<slug>-spec.md`) at `Location`. Create a dated ID directory when that is the project's convention. Colocate with the project — not in the code repo’s `docs/`.

**External work item (Linear only when registered):** pointer, not a copy. Title = the work. Description = short intent + wiki spec path/URL. Never paste the spec body. If the user started from an issue, update that issue. Otherwise create one when Product is signed off. Put `**Work:** <url>` on the spec. Omit that line when Outstanding work is wiki. `/yf-eng-build` marks the spec `implemented` and, if Linear, sets the issue **Done**.

Legacy two-file wiki specs (`product-spec.md` + `technical-spec.md`) remain valid. **New work uses this single wiki spec.**

## Always: light research subagents

Before drafting product, and again before drafting technical, launch **light** `Explore` subagents in parallel (`claude-sonnet-5`). Every run, including thin product work — do not skip because the task "looks obvious."

Typical units (merge or split to fit the work; 2–4 is usual):

- Codebase touchpoints for this change (current behavior, owners, constraints).
- Related **specs** in the wiki (parent spec, neighboring work). Outstanding **work** only in an external tracker when `PROJECT.md` registers one (`list_issues` on that Team when Destination is `linear`).
- ADRs and architecture docs in the code repo (`docs/adr/`, `docs/architecture/` when present).

Keep them light: tight question, path-cited bullets only. They inform; they do not draft the spec. When results return, Read load-bearing cited files yourself.

## Phase 1 — Product (what / why)

**Depth is a spectrum, not two modes.** Calibrate after research. State the calibration in one line when you present questions or the draft (so the user can push back).

| Signal | Lean thinner | Lean fuller |
| ------ | ------------ | ----------- |
| Origin | Named gap, bug, or follow-on to a signed spec | New capability, new users, or green field |
| "What" | Already implied by parent spec / ticket / call | Ambiguous outcome or several plausible products |
| Surface | No new journey; CLI/policy/schema tweak | New UX, new workflow, or several states |
| Forks | Few (0–2) product decisions | Scope, success, or policy is genuinely open |

**Thinner** still writes the Product half, still records decisions, still asks anything that would change the outcome — there are just fewer of those. Short sections and `N/A — <why>` are correct. Do not pad.

**Fuller** uses the whole Product template, including Experience / wireframes when UI applies.

**Clarifying questions:** ask at most **5** plan-shaping questions in the first round (0–2 for thinner work). Skip anything research already answered. Always surface **material product decisions** — as questions if the user must choose, or as proposed **Decisions** in the draft if research supports a default.

**Draft:** write the spec at the resolved destination. Fill **Product** now. Leave **Technical** as a stub (`*(Phase 2 — after product sign-off.)*`). Template:

```markdown
# <Title>

**Status:** product draft | product signed-off | technical draft | signed-off for /yf-eng-build | implemented
**Work:** <external work URL — omit when Outstanding work is wiki>

## Product

### Read this first
### Problem / opportunity
### Goals
### Users & stakeholders
### Success criteria
### Scope
#### Out of scope
#### Later
### Experience
#### Journeys & key flows
#### Design drafts / wireframes
### Constraints
### Decisions
### Assumptions & open questions
### Risks & mitigations
### Dependencies

## Technical

*(Phase 2 — after product sign-off.)*

### Context & touchpoints
### Architecture & key decisions
### Interfaces & contracts
### Data model & migrations
### Implementation sequence
### Testing strategy
### Rollout, flags & rollback
### Observability & security
### Documentation plan
### Open technical risks
### References
```

**Section roles (avoid repetition):**

- **Goals** — intent only (short, directional). No feature inventory, no acceptance tests.
- **Success criteria** — single normative ship surface. Each bullet is an *observable state that becomes true*. Do not maintain a separate In-scope list.
- **Out of scope** — hard exclusions, including intentional non-goals. One exclusion list only.
- **Later** — deferred work; not a third copy of Success criteria.
- **Constraints** — cross-cutting invariants that are not features. Do not restate Success criteria here.
- **Decisions** — what was chosen and why (including defaults from research). Even thin work lists the ones that mattered.
- Do **not** add Requirements Must/Should — those live in Success criteria (`Should:` prefix on a bullet if needed).

For non-UI work, mark **Experience** as `N/A — <why>` and skip wireframes.

**Design drafts (where UI applies):** rough wireframes in **Design drafts / wireframes**, via `claude-sonnet-5` Task/Agent subagents. ASCII or mermaid is fine — hierarchy and key states (empty, loading, error, populated), not pixel detail.

**Adversarial review:** launch one `claude-fable-5.1` `general-purpose` reviewer with the wiki spec **path only**. The reviewer verifies against codebase and docs via its own `claude-sonnet-5` `Explore` subagents, enforces **Writing for human review**, attacks the **Product** half, must not invent Technical content, and edits the file directly. Apply the acceptance rule above.

**Sign-off gate:** present the reviewed Product half and **wait for explicit sign-off**. Do not begin Phase 2 without it. On sign-off, set **Status** to `product signed-off`. If Outstanding work is `linear`, create or update the issue now (pointer only) and set `**Work:**`.

## Phase 2 — Technical (how) — only after sign-off

Re-run light research subagents as needed so the how is grounded in current code, not only Phase 1 notes.

**Draft:** fill **Technical** in the same wiki spec. Do not rewrite signed-off Product except to add a pointer (e.g. Promoted decisions later). Set **Status** to `technical draft`.

Implementation sequence (required contract for `/yf-eng-build`) — every phase:

```markdown
### Phase: <id-slug>
- **Intent:** observable state that becomes true
- **Scope:** in / explicit out
- **Depends on:** phase ids or `none`
- **Acceptance:** criteria traceable to Product success criteria
- **Test strategy:** automated / characterization / build-typecheck / other (justify)
- **Validation:** exact commands or procedures, env prerequisites, destructive/external notes
- **Done when:** checklist of observables
- **Docs expected:** none | ADR(s) | architecture concern update | AGENTS.md (as applicable)
```

Dependency order must be explicit. Validation commands must be runnable (or explicitly blocked on missing env). If research cannot fill a field, leave it as an open question — do not paper over gaps.

**Documentation plan:** which ADRs / architecture concern files / AGENTS.md updates `/yf-eng-build` should produce when decisions land (follow the target repo’s layout, e.g. `docs/adr/`, `docs/architecture/concerns/`). Binding decisions belong in the code repo as ADRs at build time — not only in this spec. Product/strategy stays in the wiki, not in the code repo.

**References** must link every source the plan relies on (parent specs, ADRs, architecture docs, code paths). The Product half is in this same document — do not point at a separate product-spec file for new work.

**Adversarial review:** launch one `claude-fable-5.1` `general-purpose` reviewer with the wiki spec **path only**. The reviewer trusts nothing in the document, verifies against code and docs via its own `claude-sonnet-5` `Explore` subagents, **edits only Technical** — Product is signed off and frozen — and enforces **Writing for human review**. Attack weak phases (missing Validation, vague Acceptance, ungrounded Docs expected, unreadable writing). Apply the acceptance rule, treating signed-off Product as an explicit user decision.

**Deliver:** present the finished spec, summarizing architecture decisions, the phased implementation sequence, docs/ADR expectations, and open technical risks needing the user's input. Set **Status** to `signed-off for /yf-eng-build` only when the user signs off the technical half (or explicitly treats the presented draft as accepted). Note that execution is `/yf-eng-build`.

## Guardrails

- Never skip the product phase, even for "obviously technical" work — a thin Product half is still required unless the user explicitly waives it.
- Never start Phase 2 before explicit user sign-off on Product.
- The spec lives in the wiki, not only in chat.
- Do not dual-write the spec into an external tracker. If Outstanding work is `linear`, the issue points at the wiki path.
- Do not put master plans or product-strategy docs into the code repo.
- Always run light research subagents (codebase, related specs, ADRs) in both phases.
- Scale Phase 1 depth to the work; do not skip material decisions because the case is simple.
