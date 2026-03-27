# Big Picture Review (BPR) v2.0

Progressive depth-first adversarial review of plans, architecture, and designs.

### What's New in v2.0

- **Modular architecture** — character, process, gates, anti-drift as separate files (was monolithic)
- **Lite mode** — quick 15-min discussions without file writes, upgradeable to full mid-session
- **Style preferences** — choose explanation depth: "more explanations" / "short and clear" / "explain when needed"
- **Research-backed critiques** — dispatches Explore agents and web searches to ground analysis in evidence
- **Participant-driven mode** — provide numbered subtopics to guide the review

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

**Two modes:**
- **Full mode** (default) — persistent review files, gate validation, checkpoints. One depth per invocation.
- **Lite mode** (`/big-picture-review lite topic`) — quick 15-min discussion with BPR's analytical character. No file writes, no gates. Say "switch to full mode" mid-session to upgrade.

Each depth level in full mode includes gate validation (automated checks for conflicts, blind spots, vision drift) before advancing.

## Features

- Persistent across sessions — resume where you left off
- Pre-mortem framing and epistemic tags ([Fact], [Judgment], [Depends])
- Anti-drift mechanisms (11 total) prevent sycophancy and scope creep
- Vision baseline — fixed at D0, never rolling, used for VISION_DRIFT detection
- Four gate modes: depth transitions, file splits, pause/resume, and pivot validation (NASA-inspired entrance checks)
- Anti-drift: three tiers — continuous (every response), periodic (checkpoints), boundary (gates)
- Dead-end handling — detects impasse, fatigue, and premature convergence
- Research-backed — dispatches Explore agents and web searches to ground critiques in evidence
- Works in English and Russian

## Installation

Copy the 5 skill files to your Claude Code skills directory:

```bash
# Personal (all projects)
mkdir -p ~/.claude/skills/big-picture-review
cp SKILL.md bpr-character.md bpr-full.md bpr-gate.md bpr-anti-drift.md ~/.claude/skills/big-picture-review/

# Or project-level
mkdir -p .claude/skills/big-picture-review
cp SKILL.md bpr-character.md bpr-full.md bpr-gate.md bpr-anti-drift.md .claude/skills/big-picture-review/
```

## Usage

```
/big-picture-review my migration to microservices
/big-picture-review critique this API design
/big-picture-review lite архитектура сайдбара
/big-picture-review quick should we use SSR here?

# Participant-driven: provide numbered subtopics in first message
/big-picture-review my API design
> 1. Auth flow  2. Rate limiting  3. Versioning strategy
```

## Requirements

- **Claude Code** (any paid plan: Pro, Max, Team, Enterprise, or API)
- **Max or Team Premium** recommended — gate validation dispatches Opus subagents, which exhaust Pro's daily budget quickly. On Pro/Team Standard, say "use Sonnet for gates" to stay within limits.
- Optional: project vision/goals document for drift detection

## What to Expect

- Creates persistent review files in `.claude/reviews/big-picture-review/`
- On first run, asks your explanation style preference (more explanations / short and clear / explain when needed)
- Gate validation dispatches Opus subagents (token-intensive). Say "use Sonnet for gates" to reduce cost.
- One depth level per invocation. Resume with the same command.
- Do not copy README.md or AGENTS.md — they are documentation only.

## Not For

- Code review (use `/quality-review-code` or similar)
- Build/test validation (use `/quality-gate` or similar)
- Implementation planning (use after BPR completes)
