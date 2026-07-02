# Forge

Shared [Claude Code](https://claude.com/claude-code) skills from Yinc, packaged as a plugin marketplace.

A *skill* is a self-contained instruction set (a `SKILL.md` file, plus any supporting files) that Claude loads on demand. This repo is both a **marketplace** (`yinc`) and the **plugin** it hosts (`forge`), which bundles every skill below. Skills are prefixed `yf-` (Yinc Forge) so their provenance is legible wherever they end up.

## Install

From a terminal:

```sh
claude plugin marketplace add yinc-studio/forge
claude plugin install forge@yinc
```

Or, inside Claude Code, run the equivalent slash commands:

```
/plugin marketplace add yinc-studio/forge
/plugin install forge@yinc
```

Once installed, skills are namespaced under the plugin and invoked as `/forge:<skill-name>` (e.g. `/forge:yf-orchestrate`).

## Skills

| Skill | Invoke | Description |
|-------|--------|-------------|
| [`yf-orchestrate`](skills/yf-orchestrate/) | `/forge:yf-orchestrate` | Act as orchestrator for the session: route each unit of work to a subagent on the best-fit model, then integrate the results. |

## Use it without installing

Every skill is a single `SKILL.md`. The simplest way to adopt one is to copy it and make it your own — drop the folder into your own skills directory and tune the routing to your models and your work:

```sh
# User-level (available in every session)
cp -R skills/yf-orchestrate ~/.claude/skills/
```

Copied this way there is no plugin namespace, so it is invoked directly as `/yf-orchestrate`.

## Updating

Bump the `version` in [`.claude-plugin/plugin.json`](.claude-plugin/plugin.json) and the matching entry in [`.claude-plugin/marketplace.json`](.claude-plugin/marketplace.json) when you change a skill, so installed users receive the update. Users refresh with:

```sh
claude plugin marketplace update yinc
```

## Adding a skill

1. Create `skills/yf-<skill-name>/SKILL.md` with YAML frontmatter (`name`, `description`) followed by the instructions.
2. Add a row to the table above.
3. Bump the plugin version (see [Updating](#updating)).

## Repo layout

```
.
├── .claude-plugin/
│   ├── marketplace.json   # marketplace manifest (name: yinc)
│   └── plugin.json        # plugin manifest (name: forge)
├── skills/
│   └── yf-orchestrate/
│       └── SKILL.md
└── README.md
```

See Anthropic's [plugin marketplace docs](https://code.claude.com/docs/en/plugin-marketplaces) for the full schema.
