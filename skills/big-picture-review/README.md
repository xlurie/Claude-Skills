# Big Picture Review (BPR)

Progressive depth-first adversarial review of plans, architecture, and designs.

## Quick Install

```bash
git clone https://github.com/xlurie/Claude-Skills.git
cp -r Claude-Skills/skills/big-picture-review ~/.claude/skills/big-picture-review
```

## What It Does

BPR challenges your thinking through structured depth levels:
- **D0:** Free dialogue — understand what you're building, why, for whom
- **D1:** High-level approach evaluation — compare alternatives, find trade-offs
- **D2+:** Deep dives — edge cases, implementation details, concrete parameters

Each depth level includes gate validation (automated checks for conflicts, blind spots, vision drift) before advancing.

## Features

- Persistent across sessions — resume where you left off
- Pre-mortem framing and epistemic tags ([Fact], [Judgment], [Depends])
- Anti-drift mechanisms (11 total) prevent sycophancy and scope creep
- Vision baseline tracking with VISION_DRIFT detection
- Gate system with NASA-inspired entrance checks

## Installation

Copy the 3 skill files to your Claude Code skills directory:

```bash
# Personal (all projects)
mkdir -p ~/.claude/skills/big-picture-review
cp SKILL.md bpr-gate.md bpr-anti-drift.md ~/.claude/skills/big-picture-review/

# Or project-level
mkdir -p .claude/skills/big-picture-review
cp SKILL.md bpr-gate.md bpr-anti-drift.md .claude/skills/big-picture-review/
```

## Usage

```
/big-picture-review my migration to microservices
/big-picture-review critique this API design
```

## Requirements

- **Claude Code** (any paid plan: Pro, Max, Team, Enterprise, or API)
- **Max or Team Premium** recommended — gate validation dispatches Opus subagents, which exhaust Pro's daily budget quickly. On Pro/Team Standard, say "use Sonnet for gates" to stay within limits.
- Optional: project vision/goals document for drift detection

## What to Expect

- Creates persistent review files in `.claude/reviews/big-picture-review/`
- Gate validation dispatches Opus subagents (token-intensive). Say "use Sonnet for gates" to reduce cost.
- One depth level per invocation. Resume with the same command.

## Not For

- Code review (use `/quality-review-code` or similar)
- Build/test validation (use `/quality-gate` or similar)
- Implementation planning (use after BPR completes)
