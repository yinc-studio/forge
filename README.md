# Forge

Shared agent skills from Yinc, packaged for **Claude Code** and **Cursor**.

A *skill* is a self-contained instruction set (a `SKILL.md` file, plus any supporting files) that the client loads on demand. This repo is both a **marketplace** (`yinc`) and the **plugin** it hosts (`forge`). Skills are prefixed `yf-` (Yinc Forge).

Each client has its own plugin directory with its own manifest and skill bodies. Behavior matches; only model IDs and agent-type names differ.

| Role | Claude (`claude/skills/`) | Cursor (`cursor/skills/`) |
|------|---------------------------|---------------------------|
| Research / simple build | `claude-sonnet-5` | `cursor-grok-4.5-high` |
| Complex build | `claude-opus-5` | `gpt-5.6-terra-medium` |
| Plan / writing / code review | `claude-fable-5` | `claude-fable-5-thinking-high` |
| Fast / wiki lookup | `claude-haiku-4-5` | `cursor-grok-4.5-high` |

## Install

### Claude Code

```sh
claude plugin marketplace add yinc-studio/forge
claude plugin install forge@yinc
```

Or, inside Claude Code:

```
/plugin marketplace add yinc-studio/forge
/plugin install forge@yinc
```

The Claude marketplace (`.claude-plugin/marketplace.json`) points at `./claude`. Skills invoke as `/forge:yf-eng-plan`, etc.

### Cursor

Add `yinc-studio/forge` as a team / plugin marketplace, then install the `forge` plugin. The Cursor marketplace (`.cursor-plugin/marketplace.json`) points at `./cursor`.

Invoke skills as `/yf-eng-plan`, `/yf-eng-build`, etc.

## Skills

| Skill | Invoke | Description |
|-------|--------|-------------|
| [`yf-orchestrate`](claude/skills/yf-orchestrate/) | `/yf-orchestrate` | Act as orchestrator for the session: route each unit of work to a subagent on the best-fit model, then integrate the results. |
| [`yf-research-market`](claude/skills/yf-research-market/) | `/yf-research-market` | Breadth-first market validation scan for a business idea: parallel lane research, structured findings, and a depth menu. |
| [`yf-eng-plan`](claude/skills/yf-eng-plan/) | `/yf-eng-plan` | Two-phase software planning: product-spec → (sign-off) → technical-spec with a phased `/yf-eng-build` contract. |
| [`yf-eng-build`](claude/skills/yf-eng-build/) | `/yf-eng-build` | Execute a signed-off technical-spec with phased test-first implementation, observed validation, and ADR/architecture updates. |

Claude bodies: [`claude/skills/`](claude/skills/). Cursor counterparts: [`cursor/skills/`](cursor/skills/).

## Use it without installing

```sh
# Claude Code (user-level)
cp -R claude/skills/yf-eng-plan ~/.claude/skills/

# Cursor (user-level)
cp -R cursor/skills/yf-eng-plan ~/.cursor/skills/
```

Copied this way there is no plugin namespace, so it is invoked directly as `/yf-eng-plan`.

## Updating

Bump `version` in both plugin manifests and both marketplace entries when you change a skill:

- [`claude/.claude-plugin/plugin.json`](claude/.claude-plugin/plugin.json)
- [`cursor/.cursor-plugin/plugin.json`](cursor/.cursor-plugin/plugin.json)
- [`.claude-plugin/marketplace.json`](.claude-plugin/marketplace.json)
- [`.cursor-plugin/marketplace.json`](.cursor-plugin/marketplace.json)

Claude users refresh with:

```sh
claude plugin marketplace update yinc
```

## Adding a skill

1. Create **both** `claude/skills/yf-<name>/SKILL.md` (exact Claude model IDs) and `cursor/skills/yf-<name>/SKILL.md` (Cursor Task allowlist slugs).
2. Keep behavior identical; only model IDs, agent-type names, and client-specific tool wording should differ.
3. Add a row to the table above.
4. Bump the plugin version (see [Updating](#updating)).

## Repo layout

```
.
├── .claude-plugin/
│   └── marketplace.json          # marketplace (name: yinc) → source ./claude
├── .cursor-plugin/
│   └── marketplace.json          # marketplace (name: yinc) → source ./cursor
├── claude/
│   ├── .claude-plugin/
│   │   └── plugin.json           # Claude forge plugin
│   └── skills/
│       ├── yf-orchestrate/
│       ├── yf-research-market/
│       ├── yf-eng-plan/
│       └── yf-eng-build/
├── cursor/
│   ├── .cursor-plugin/
│   │   └── plugin.json           # Cursor forge plugin
│   └── skills/
│       ├── yf-orchestrate/
│       ├── yf-research-market/
│       ├── yf-eng-plan/
│       └── yf-eng-build/
└── README.md
```
