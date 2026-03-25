# Claude Skills

A collection of custom skills for [Claude Code](https://docs.anthropic.com/en/docs/claude-code). Clone the repo and copy any skill to your Claude Code skills folder to use it.

```bash
git clone https://github.com/xlurie/Claude-Skills.git
cp -r Claude-Skills/skills/big-picture-review ~/.claude/skills/big-picture-review
```

## Skills

| Skill | Description |
|-------|-------------|
| [Big Picture Review](skills/big-picture-review/) | Progressive depth-first adversarial review of plans, architecture, and designs. Anti-drift mechanisms, gate validation, persistent across sessions. |

## Installation

Each skill has its own directory under `skills/`. Copy the skill directory to your Claude Code skills folder:

```bash
# Personal (all projects)
cp -r skills/big-picture-review ~/.claude/skills/big-picture-review

# Or project-level
cp -r skills/big-picture-review .claude/skills/big-picture-review
```

See each skill's README for details.

## License

MIT
