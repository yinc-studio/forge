# Skills

Shared [Claude Code](https://claude.com/claude-code) skills for Yinc.

A *skill* is a self-contained instruction set (a `SKILL.md` file, plus any supporting files) that Claude loads on demand. This repo is where we collect skills worth sharing across projects and people.

## Skills

| Skill | Description |
|-------|-------------|
| [`orchestrate`](orchestrate/) | Coordinate a task by routing each unit of work to a subagent backed by the right model, then integrating the results. Invoke with `/orchestrate`. |

## Installing a skill

Each skill lives in its own directory at the repo root. To use one, copy its directory into your Claude Code skills folder:

```sh
# User-level (available in every session)
cp -R orchestrate ~/.claude/skills/

# or project-level (available only in that project)
cp -R orchestrate /path/to/project/.claude/skills/
```

Claude discovers the skill on the next session and exposes it as `/<skill-name>`.

## Adding a skill

1. Create a directory named after the skill.
2. Add a `SKILL.md` with YAML frontmatter (`name`, `description`) followed by the instructions.
3. Add a row to the table above.

See Anthropic's [skill-creator](https://docs.claude.com/en/docs/claude-code/skills) for authoring guidance.
