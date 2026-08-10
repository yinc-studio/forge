---
name: yf-orchestrate
description: "Coordinate the work by routing each piece of a task to a subagent backed by the right model, then integrating the results. Use when the user runs /yf-orchestrate, asks to orchestrate subagents, or wants work delegated and managed across multiple specialized agents."
---

# Orchestrate

Act as the orchestrator for the rest of the session. Decompose the task, delegate each piece to a subagent backed by the model best suited to it, and integrate the results. These routing rules stay in effect for the **entire session**, not just the first task.

## Operating loop

1. Break the task into discrete units of work (write code, review code, search the wiki, web search, summarize, create or review writing/plans).
2. For each unit, launch a subagent via the Task/Agent tool with the model from the routing table below. Pass the `model` parameter explicitly (exact ID from the table — never silently substitute), and set the reasoning effort to `high`.
3. Run independent units in parallel — batch multiple Task/Agent calls in one message.
4. Review each subagent's output before integrating. You own the final result.
5. Repeat until the task is complete.

## Models (exact Claude model IDs)

| Work | `model` | Effort |
|------|---------|--------|
| Simple code generation | `claude-sonnet-5` | high |
| Searching for content in the wiki | `claude-haiku-4-5` | high |
| Web searches | `claude-sonnet-5` | high |
| Summarizing content | `claude-haiku-4-5` | high |
| Reviewing code | `claude-fable-5` | high |
| Reviewing plans | `claude-fable-5` | high |
| Reviewing writing | `claude-fable-5` | high |
| Writing (general) | `claude-sonnet-5` | high |
| Planning, plan documents | `claude-fable-5` | high |
| Complex code generation | `claude-opus-5` | high |

### Routing notes

- **Simple vs. complex code**: route to `claude-opus-5` when the change spans multiple files/systems, needs non-trivial design, or has tricky logic. Otherwise use `claude-sonnet-5`.
- **Create vs. review**: creating and reviewing writing or plans both go to `claude-fable-5`. Pair them — have one subagent create, another review, when quality matters.
- If a unit doesn't map cleanly to a row, pick the closest match and note the choice.

## Delegation rules

- Each subagent starts fresh with no access to the conversation. Give it a self-contained prompt: full context, the exact deliverable, and what to return.
- Specify precisely what the subagent should return so its output integrates cleanly.
- Launch parallel subagents in a single message when units are independent; sequence them only when one depends on another's output.
