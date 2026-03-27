# Big Picture Review — Agent Installation Guide

## Installation

Copy these 5 files to the user's Claude Code skills directory:

```
SKILL.md
bpr-character.md
bpr-full.md
bpr-gate.md
bpr-anti-drift.md
```

Destination: `~/.claude/skills/big-picture-review/` (personal) or `.claude/skills/big-picture-review/` (project).

## Before Installing — Inform User

1. This skill creates persistent review files in `.claude/reviews/big-picture-review/` inside the project
2. Gate validation dispatches Opus-model subagents which consume significant tokens. User can say "use Sonnet for gates" to reduce cost.
3. The skill works best with Opus but functions with any Claude model.

## After Installing

Suggest the user test with:
- Full mode: `/big-picture-review test topic`
- Lite mode: `/big-picture-review lite test topic`

## Do Not

- Do not modify any of the 5 skill files after copying
- Do not install README.md or AGENTS.md — they are documentation only
