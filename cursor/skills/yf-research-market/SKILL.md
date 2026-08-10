---
name: yf-research-market
description: Run a breadth-first market validation scan for a business idea by orchestrating parallel research subagents. Use whenever the user shares a business or product idea and wants to know what else is out there, who the competitors are, whether the market is crowded, or whether the idea is worth pursuing — even if they don't say "research" explicitly. Phrases like "is anyone doing this", "validate this idea", "scan the market", "what's the competitive landscape", or "/yf-research-market" should all trigger this skill. Do not use for deep single-company research or general web questions.
---

# Market Research (Breadth-First Validation Scan)

Quickly map the landscape around a business idea so the user can judge whether it merits deeper exploration. Optimize for **breadth first**: cover the whole territory cheaply and comparably, then go deep only where the user directs. The output is evidence and open questions — never a go/no-go verdict.

## Design principles

These govern every step; when in doubt, resolve toward them.

1. **Breadth is only useful if structured.** Every research lane returns the same schema. Unstructured findings force the synthesis step to pay for untangling; structured findings make lanes comparable and synthesis cheap.
2. **Cheap models for extraction, strong models for judgment.** Lane research is extraction work — finding names, pricing, positioning. Use the cheapest capable model there. Reserve the strong model for decomposition and synthesis.
3. **Hard budgets everywhere.** Each lane agent gets a fixed search budget. Cost discipline comes from caps, not from hoping agents are frugal.
4. **Evidence, not verdicts.** The user has not defined go/no-go criteria. A confident "skip this market" is fake precision. Surface claims + evidence + confidence + open questions, and let the user judge.
5. **Depth is user-directed.** The breadth pass ends with a menu of depth questions. Never launch a depth pass unprompted.

## Model routing

- **Lane agents (Step 2):** Launch one Task subagent per lane in parallel with `model: cursor-grok-4.5-high` and `subagent_type: generalPurpose`. Pass `model` and `subagent_type` explicitly on every Task call.
- **Depth pass (Step 4):** `gpt-5.6-terra-medium` with `subagent_type: generalPurpose`.
- **Steps 0, 1, and 3** run on the orchestrating agent — do not delegate decomposition or synthesis to subagents.

## Pipeline

### Step 0 — Frame the idea (no searches)

Restate the idea in one or two sentences: the job being done, for whom, and the rough form of the solution. If the idea is too vague to generate meaningful lanes (no identifiable customer or job), ask one clarifying question before proceeding. Otherwise don't ask — proceed and state your framing assumptions at the top of the output.

### Step 1 — Decompose into lanes (strong model, one call)

Generate 5–7 research lanes. Always include these five; add up to two idea-specific lanes if clearly warranted:

1. **Direct competitors** — products solving the same job for the same customer.
2. **Adjacent & substitute solutions** — different product categories solving the same underlying job.
3. **Incumbent overlap** — large platforms whose existing footprint means they could add this as a feature. Note actual signals (product pages, announcements, job postings), not just theoretical capability.
4. **Status quo** — how the target customer solves this today with no product at all: spreadsheets, human advisors, communities, doing nothing. This lane is mandatory and often decisive; the real competitor is frequently "nothing."
5. **Market motion** — recent entrants, funding events, shutdowns, and pivots in the category (roughly last 2–3 years). Whether new entrants are getting funded or dying is a stronger signal than the static competitor list.

Cap at 7 lanes total. If decomposition wants more, merge — more lanes past this point add cost faster than insight.

### Step 2 — Fan out lane agents (cheap model, parallel)

Spawn one subagent per lane, all in the same turn so they run in parallel. Each agent's charter:

- **Scope:** the single lane, nothing else. If it stumbles on findings for another lane, it may include them but must not spend budget chasing them.
- **Search budget:** 3–5 searches, hard cap. Prefer 3; use 5 only if early results are thin.
- **Snippet-first policy:** work from search snippets; fetch a full page only when a snippet is ambiguous on a schema field (e.g., pricing unclear). Cap full-page fetches at 2 per agent.
- **Recency:** prefer sources from the last 2 years for market-motion claims; timeless is fine for "what the company does."
- **Output:** JSON only, conforming to the schema below. No prose narration.

**Per-finding schema** (each agent returns a JSON array of these):

```json
{
  "name": "Company / solution / behavior name",
  "lane": "direct | adjacent | incumbent | status-quo | motion",
  "what_it_does": "One or two sentences, concrete",
  "target_customer": "Who it's for, as specifically as visible",
  "pricing_model": "e.g. $20/mo subscription, % AUM, free, unknown",
  "traction_signal": "Funding, user counts, growth/decline evidence, or 'none found'",
  "recency": "How current the evidence is",
  "confidence": "high | medium | low",
  "sources": ["url", "url"]
}
```

Empty lanes are findings too: an agent that finds little should say so explicitly ("searched X, Y, Z; no direct competitors surfaced") rather than padding with weak matches.

**If subagents are unavailable** (e.g., running in a plain chat context): run the lanes yourself, sequentially, under the same budgets and schema. The discipline matters more than the parallelism.

### Step 3 — Synthesize (strong model, one call)

Merge all lane outputs, dedupe entities appearing in multiple lanes (keep the union of their evidence), then produce the report below. Ground every claim in the lane findings; if lanes conflict, say so rather than resolving silently.

## Report structure

Use this exact template:

```
# Market scan: [idea, one line]

**Framing assumptions:** [how the idea was interpreted; 1–2 sentences]

## Market map
[Position the findings along 1–2 axes that actually differentiate this
market — e.g., DIY↔full-service, horizontal↔job-specific. Choose axes
from the evidence, not a stock 2x2. Prose or a compact table.]

## Notable signals
[3–6 bullets. Each: claim + supporting evidence + confidence. Candidates:
crowdedness, incumbent gravity, pricing patterns, funding momentum or
graveyard, strength of the status quo, whitespace. Only include signals
the evidence actually supports.]

## What would decide this
[3–5 questions that would actually determine whether the idea is worth
pursuing. For each: why it's decisive, and what a depth pass would need
to do to answer it (sources, effort). This is the depth menu.]

## Coverage notes
[What the scan did NOT cover, lanes that came back thin, and any
low-confidence areas. Honesty here is what makes the breadth pass
trustworthy.]
```

Keep the report tight — the value is in comparability and the depth menu, not length.

## Step 4 — Depth pass (only on user direction)

When the user picks a question from the depth menu, run a single focused agent (or do it directly) with a larger budget: up to 10 searches and 5 full-page fetches, strong model allowed since depth work involves judgment. Output format: answer the chosen question specifically, with the same claims + evidence + confidence discipline, ending with any new questions the answer surfaced. Do not re-scan breadth.

## Cost levers (tune here, in order of impact)

1. Model tiering — cheap model for lanes is non-negotiable unless the user overrides.
2. Search caps per lane agent (default 3–5).
3. Snippet-first with fetch caps (default 2 per lane agent).
4. Lane cap (default 7).

If the user asks for a "cheaper" or "faster" scan, drop to the five mandatory lanes at 3 searches each. If they ask for a "thorough" scan, raise search caps to 6–8 per lane before adding lanes.
