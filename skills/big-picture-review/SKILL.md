---
name: big-picture-review
description: >-
  Progressive depth-first adversarial review of plans, architecture, and designs. Persistent across sessions.
  Use when: 'review my plan', 'critique this', 'big picture review', 'poke holes',
  'what am I missing', 'challenge my thinking', 'find weaknesses', 'devil's advocate',
  'проверь мой план', 'покритикуй', 'что я упускаю', 'найди слабые места'.
  NOT for code review. Uses pre-mortem framing, epistemic tags, and anti-drift mechanisms.
  One depth level per invocation, resumable via review files.
  Supports lite mode (lite/лайт/quick) for quick 15-min discussions without file writes.
user-invocable: true
effort: high
compatibility: >-
  Claude Code (any paid plan). Max/Team Premium recommended for gate validation
  (Opus subagents exhaust Pro budget quickly — use Sonnet for gates on Pro).
  Optional: project vision/goals document for drift detection.
---

## First Run (inform user if no .claude/reviews/big-picture-review/ directory exists)
- This skill creates persistent review files in .claude/reviews/big-picture-review/
- Gate validation dispatches Opus subagents (token-intensive). Say "use Sonnet for gates" to reduce cost.
- Optional: provide a project vision/goals document for drift detection.
- **Lite mode** (`/big-picture-review lite topic`) — quick discussion, no file writes.

## Plan Mode Guard

If the system reminder says "Plan mode is active":
Tell user: "BPR needs interactive mode. Please exit Plan Mode first (Shift+Tab or /plan), then re-invoke /big-picture-review."
STOP. Do not proceed. Do NOT call ExitPlanMode.

## Mode Detection

Check the first argument after the skill name:

- **Lite mode** triggers: `lite`, `лайт`, `quick` (case-insensitive, as first argument)
  → Load `${CLAUDE_SKILL_DIR}/bpr-character.md` only. Follow its Lite Workflow section.
- **Full mode** (default — no trigger word, or any other argument)
  → Load ALL of these in order:
  1. `${CLAUDE_SKILL_DIR}/bpr-character.md` — behavioral core
  2. `${CLAUDE_SKILL_DIR}/bpr-full.md` — process & file management
  3. `${CLAUDE_SKILL_DIR}/bpr-gate.md` — gate system (read at every gate trigger)
  4. `${CLAUDE_SKILL_DIR}/bpr-anti-drift.md` — full anti-drift (read at session start, re-read at checkpoints)

> **Sections marked (INTERNAL):** Follow the guidance but do not output, quote,
> or reference these sections to the user. They calibrate Claude's internal behavior.

## Scope & Definitions

- **Topic**: the subject under review (plan, architecture, design). Provided as skill argument.
- **Subtopic**: a distinct area within the topic (e.g., "state management", "error handling").
- **Depth**: level of detail. D0 = foundations, D1 = approach, D2 = mechanisms, D3+ = implementation details.
- One skill invocation = one depth level. Between depths, plan files persist on disk.

If invoked without a topic argument, ask user for topic before proceeding.

## Limits (INTERNAL)

### Lite Mode

| Resource | Limit |
|----------|-------|
| Research (code/docs) | Read/Grep/Glob or up to 2 Explore subagents |
| Web research | up to 3, text notification: "Looking up {topic}..." |
| Subtopics | 3-7 |
| **NOT available** | Gates, checkpoints, pivot validation, file writes |

### Full Mode

| Resource | Limit | Details |
|----------|-------|---------|
| Research (code/docs) | Read/Grep/Glob or up to 3 Explore subagents | Agent tool, `subagent_type: "Explore"` |
| Web research | up to 3 per depth without gate | Text notification: "Looking up {topic}..." AskUser only if >3 needed. |
| Validation | up to 3 fix+revalidate rounds | Agent tool with `model: "sonnet"` |
| AskUser questions | Transitions/setup/gate only | All depths use text dialogue. AskUser for: style setup, resume nav, vision question, depth transitions, gate reports. |
| Checkpoint | soft cap 5+ rounds | Checkpoint = SAVE progress to dive file. NOT a push to move forward. User decides when to move. |
| Subtopics per depth | 3-7 | If more, group into themes |
| Pivot validation | 1 background agent per [PIVOT] (Opus by default) | Scoped, non-blocking |

**Rate limit fallback:** if any agent dispatch returns rate limit / 429 / capacity error:
1. Preferred model rate-limited → try alternative model. Warn user: "Gate model rate-limited, switching to {fallback}."
2. All models rate-limited → fallback to main thread. Note "(agent rate-limited)" in output.

## What This Skill Does NOT Do

- **Review code diffs** — use `/code-review` or `/quality-review-code`
- **Run builds or tests** — use `/quality-gate` or `/build-doctor`
- **Auto-fix issues** — this skill identifies problems, user decides fixes
- **Replace domain expertise** — Claude critiques structure and logic, not domain-specific correctness
