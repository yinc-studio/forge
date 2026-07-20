---
name: yf-orchestrate
description: "Coordinate the work by routing each piece of a task to a subagent backed by the right model, then integrating the results. Use when the user runs /yf-orchestrate, asks to orchestrate subagents, or wants work delegated and managed across multiple specialized agents."
---

# Orchestrate

Act as the orchestrator for the rest of the session. Decompose the task, delegate each piece to a subagent backed by the model best suited to it, and integrate the results. These routing rules stay in effect for the **entire session**, not just the first task.

## Operating loop

1. Break the task into discrete units of work (write code, review code, search the wiki, web search, summarize, create or review writing/plans).
2. For each unit, launch a subagent via the Task tool with the model from the routing table below. Pass the `model` parameter explicitly.
3. Run independent units in parallel — batch multiple Task calls in one message.
4. Review each subagent's output before integrating. You own the final result.
5. Repeat until the task is complete.

## Routing table

Map each unit of work to a model. Always pass the listed slug as the Task tool `model` parameter.

| Work | Model slug |
|------|-----------|
| Advisor | `fable` |
| Simple code generation | `sonnet` |
| Searching for content in the wiki | `haiku` |
| Web searches | `sonnet` |
| Summarizing content | `haiku` |
| Reviewing code | `opus` |
| Reviewing plans | `fable` |
| Reviewing writing | `opus` |
| Writing (general) | `sonnet` |
| Planning, plan documents | `fable` |
| Complex code generation | `opus` |

### Routing notes

- **Simple vs. complex code**: route to `opus` when the change spans multiple files/systems, needs non-trivial design, or has tricky logic. Otherwise use `sonnet`.
- **Create vs. review**: creating and reviewing writing or plans both go to `fable`. Pair them — have one subagent create, another review, when quality matters.
- **Advisor**: Use as a sounding board to pressure-test your own reasoning on hard, ambiguous, or high-stakes decisions — architectural trade-offs, debugging dead-ends, conflicting requirements, or choosing between approaches. Hand it your current thinking (the problem, the options you're weighing, and your tentative conclusion) and ask it to challenge assumptions, surface blind spots, and flag risks. It advises only; it does not produce the deliverable. Skip it for routine or unambiguous work. Only use when the root-level agent is opus or sonnet.
- If a unit doesn't map cleanly to a row, pick the closest match and note the choice.

## Delegation rules

- Each subagent starts fresh with no access to the conversation. Give it a self-contained prompt: full context, the exact deliverable, and what to return.
- Specify precisely what the subagent should return so its output integrates cleanly.
- Launch parallel subagents in a single message when units are independent; sequence them only when one depends on another's output.
