# BPR Gate System

> **When to read:** At every gate trigger (split, transition, pause, pivot).
> This file is the SINGLE SOURCE for gate mechanics. SKILL.md references this file.

## Gate Modes

| Mode | Trigger | Agent | Scope | Exits |
|------|---------|-------|-------|-------|
| Split | dive >300 lines | <5 decisions: Main inline / 5+: scoped Sonnet | current depth, differential | continue |
| Transition | D→D+1 | Opus (or user-specified model), full context | all depths, absolute | Go / Recycle / Kill |
| Pause | user says stop | Opus (or user-specified model), relaxed entrance | all depths, absolute | save + exit |
| Pivot | [PIVOT] decision | Opus (or user-specified model), background | scoped to pivot | integrate at checkpoint |

## Gate Entrance Check (NASA-inspired)

Before dispatching any gate agent, verify ALL 4 conditions. If any fail → fill gaps first, do NOT dispatch.

1. **Dive file exists** and is non-empty
2. **All subtopics** have status recorded (covered / skipped / incomplete)
3. **All decisions** have rationale recorded (not just labels)
4. **master.md** updated with current depth summary

**Relaxed entrance (Pause mode only):** Conditions 2-3 are advisory — record gaps but proceed anyway. Rationale: user stopping shouldn't be blocked by incomplete state.

## Agent Prompt Template

Use this template for ALL gate agent dispatches. Replace placeholders with actual content.

```
You are a BPR gate validation agent. Mode: {mode}.

## Context
{context_bullets}

## Check List
Run ONLY these checks (mode-specific):
{check_list}

## Source Material
Review ALL of the following in full (do NOT summarize or skip):

### master.md
{master_md}

### Dive Files
{dive_files}

## Output Format
For each finding, output:
[CHECK_TYPE] [severity: Critical/High/Medium/Minor] [confidence: 0.0-1.0] {reference}
- kind: fail | review | mechanical
- condition: what IS (verbatim quote from source, MANDATORY — no quote = confidence drops to 0.3)
- criteria: what SHOULD BE
- evidence: verbatim quote proving the condition
- consequence: impact if not fixed

Group findings by CHECK_TYPE. Sort groups by highest severity within group.
End with: GATE RESULT: {GO|RECYCLE|KILL} with 1-sentence justification.
```

**Template rules:**
- `{dive_files}` = full text of ALL dive files. Always full text, never summaries (model context is sufficient).
- `{context_bullets}` = 3-5 bullets from Main: undocumented themes, user preferences, exploration direction.
- `{check_list}` = the applicable checks from the per-mode lists below.
- `{mode}` = Split | Transition | Pause | Pivot.

## Per-Mode Check Lists

### Split Mode Checks
When dive file exceeds 300 lines:
- **CONFLICTS** — current depth only, differential (new decisions since last check)
- **WEAK_DECISIONS** — current depth only, differential
- **STRUCTURAL** — current depth only (circular deps, orphaned decisions, duplicates)
- **MECHANICAL** — current depth only (typos, numbering, conventions)
- **CARRY_FORWARD** — MANDATORY: verify new split part opens with carry-forward block
- If <5 decisions: Main runs these inline (no agent dispatch)
- If 5+ decisions: dispatch scoped Sonnet agent

### Transition Mode Checks
When moving D→D+1 (full validation):
- **CONFLICTS** — ALL depths, absolute (direct / resource / philosophical)
- **CONDITIONAL_CONFLICTS** — ALL depths (A+B ok, A+B+C not)
- **DEPENDS_UNVERIFIED** — ALL depths (unresolved [Depends] tags, assumption drift)
- **WEAK_DECISIONS** — ALL depths, absolute (single-option bias, weak rationale, wishful thinking)
- **BLIND_SPOTS** — ALL depths (topics mentioned but not critiqued, skipped subtopics)
- **STRUCTURAL** — ALL depths (circular deps, orphaned, duplicates, scope creep, superseded-unmarked)
- **MECHANICAL** — ALL depths (typos, numbering, conventions, duplicate text)
- **VISION_DRIFT** — MANDATORY (compare decisions against "Vision Baseline" section in master.md, fixed at D0, not rolling). Skip if "Vision source: none".
- **COS_CHECK** — MANDATORY ("Argue that drift HAS occurred. Evaluate your own evidence.")
- **SECOND_ORDER** — ALL depths ("and then what?" — consequences of consequences)
- **ASSUMPTION_DRIFT** — ALL depths (assumptions held since start, not recently validated)
- **PROJECT_SPECIFIC** — via Main's context bullets (e.g., domain constraints, regulatory requirements, tech limitations)
- **BREAKTHROUGH** — Main generates after receiving agent results (not part of agent checklist)

### Pause Mode Checks
When user says stop/enough/pause:
- **CONFLICTS** — ALL depths, absolute
- **CONDITIONAL_CONFLICTS** — ALL depths
- **DEPENDS_UNVERIFIED** — ALL depths
- **WEAK_DECISIONS** — ALL depths, absolute
- **BLIND_SPOTS** — current depth only (reduced scope — user is leaving)
- **STRUCTURAL** — ALL depths
- **MECHANICAL** — ALL depths
- **VISION_DRIFT** — MANDATORY
- **ASSUMPTION_DRIFT** — ALL depths
- **ISSUES_AT_PAUSE** — RECORD: capture state snapshot for resume (what's open, what's at risk)

**Pause-specific rules:**
- Relaxed entrance (see above)
- Critical findings do NOT block pause — record prominently, don't trap user
- Pause findings → ISSUES_AT_PAUSE field in master.md

### Pivot Mode Checks
When a decision invalidates 2+ previous decisions ([PIVOT] tag):
- **CONFLICTS** — scoped to pivot decision and its affected decisions
- **STRUCTURAL** — scoped (dependency graph around pivot)
- **MECHANICAL** — current depth only
- **IMPLICIT_KNOWLEDGE** — MANDATORY: what did the old approach know/solve implicitly that the new approach must account for?
- **HALT_WATCH** — MANDATORY: triage affected decisions into:
  - HALT: in-flight work that will become wrong (stop immediately)
  - WATCH: planned work that hasn't started (reassess before starting)
- **SECOND_ORDER** — pivot consequences only
- **ASSUMPTION_DRIFT** — scoped to pivot

**Pivot-specific rules:**
- Dispatch in BACKGROUND — don't block discussion momentum
- When agent returns: integrate findings at next checkpoint
- Blast radius mapping via dependency graph (which decisions reference the pivoted ones?)

## Agent Output Format (SARIF-inspired)

Each finding MUST include:

```
[CHECK_TYPE] [severity] [confidence: 0.0-1.0] {reference to dive file + line/section}
- kind: fail | review | mechanical
  - fail: must resolve before gate opens
  - review: worth discussing, user decides
  - mechanical: auto-fixable (typos, numbering, formatting)
- condition: what IS (current state, verbatim quote MANDATORY)
- criteria: what SHOULD BE (expected state per audit 5C framework)
- evidence: verbatim quote from dive file proving the condition
- consequence: impact if not fixed (concrete, not vague)
```

**Rules:**
- No quote in evidence → confidence automatically drops to 0.3
- Confidence thresholds: ≥0.8 = high confidence, 0.5-0.8 = medium, <0.5 = low (present but flag uncertainty)
- Group findings by CHECK_TYPE, sort groups by highest severity within group

## Transition Gate Flow (7 Steps)

1. **SAVE** — Save current dive file (complete checkpoint)
2. **ANNOUNCE** — Tell user: "Running transition gate validation before D{N+1}..."
3. **DISPATCH** — Send agent (Opus by default, or user-specified model) with: all dive files (full text) + master.md + Main's 3-5 context bullets
4. **RECEIVE** — Structured findings in SARIF-inspired format
5. **FILTER** — Main validates findings:
   - Auto-fix MECHANICAL errors silently (show summary count only)
   - Drop false positives (explain why if dropping ≥2)
   - Validate severity assignments
6. **PRESENT** — Gate Report (see presentation format below)
7. **MARK** — Update master.md: `Validation: done|pending|recycle ({date}) | Result: GO|RECYCLE|KILL | Findings: {summary}`

## Gate Report Presentation (D38)

After gate completes, Main presents results:

1. **Full report in ONE message** — summary table (counts by severity) + all findings grouped by CHECK_TYPE. Do NOT split across multiple messages.
2. **After report:** AskUser with navigation choice:
   - "High severity first" — walk through Critical + High findings
   - "All groups" — walk through every group sequentially
   - "Only High + open questions" — Critical + High + unresolved questions
3. **Then:** walk through selected groups ONE AT A TIME. AskUser per group for user decisions.
4. **Never:** dump findings without follow-up questions. Never split report across messages without navigation.

This prevents: (a) wall of text without actionable steps, (b) findings lost between messages, (c) user overwhelm.

## Findings Handling (3 Streams)

### Stream 1: MECHANICAL
Auto-fix silently. Show summary only: "Fixed {N} mechanical issues (typos, numbering)."
No user interaction needed.

### Stream 2: DIRECT CONFLICTS (Critical/High)
Must resolve before gate opens:
- Present conflict with both sides + evidence
- Show approaches with trade-offs
- User decides
- If unresolvable → Recycle gate exit

### Stream 3: EVERYTHING ELSE
Resource conflicts, philosophical tensions, structural issues, blind spots, weak decisions, plus Breakthrough question:
- Group by relatedness (not by CHECK_TYPE — related findings across types go together)
- Sort groups by highest severity within group
- Lightweight BPR per group (critique + approaches + user decision)
- After all groups: final contradiction check between NEW decisions made during gate resolution

### Where resolved findings go:
- **Conflicts** → update original dive files (add supersession links)
- **New decisions** → beginning of new dive file as "Gate findings resolved" section
- **Accepted risks** → risk register in master.md

## Gate Exits (Cooper Stage-Gate)

### Go
All Critical findings resolved. Proceed to D+1.
- Freeze baseline: decisions from this depth become the reference point
- Minor/Medium unresolved → carry forward as known issues

### Recycle
Critical findings unresolvable at current depth → return to D{N}, reopen affected subtopics.
- Identify which subtopics need reopening
- Don't restart entire depth — targeted revisit

### Kill
Approach fundamentally unviable → close review.
- Write final summary with reasons
- Record what was learned (not wasted — informs next attempt)

## Baseline Freeze (Severity-Based)

After a Go exit, the depth's decisions become the baseline:

- **Minor/Medium changes** to frozen decisions: soft override allowed
  - Record rationale in current dive file
  - Note override in master.md
- **High/Critical changes** to frozen decisions: formal Recycle through gate
  - Cannot override — must revalidate

**Escalation rules:**
- If override requires changes to 2+ OTHER decisions → Recycle, not override
- Agent tracks: 3+ soft overrides to the SAME decision → flag "consider formal revision"

## Pause Gate Specifics

### Save Protocol (SBAR format)
After Pause gate checks complete, save in SBAR format:
- **Situation:** Current state of review (depth, subtopic, what's open)
- **Background:** What led here (key decisions, user's direction)
- **Assessment:** Claude's evaluation of current state + ISSUES_AT_PAUSE from gate
- **Recommendation:** Specific next action for resume (not generic "continue review")
- **Plus:** Assumption audit (what assumptions haven't been validated), failed approaches log, timestamp all state

### Resume Protocol
When returning to a paused review:
1. **External change scan:** Have files in review directory changed since last save?
2. **Staleness classification:**
   - time-only: just time passed, no changes
   - change-triggered: files were modified externally
   - hard miss: key assumptions invalidated by external changes
   - assumption-violated: something we assumed turned out wrong
3. **SBAR summary:** Show user the saved SBAR from pause
4. **Synthesis confirmation (I-PASS):** User acknowledges current state before continuing
5. **Re-ask style preference** (even if stored in master.md — preferences may change)
6. **Pending gate check:** If master.md shows `Validation: pending` → run gate first before resuming depth

## Pivot Gate Specifics

### Trigger
A decision that invalidates 2+ previous decisions gets tagged [PIVOT] in the dive file.

### Flow
1. **MARK** as [PIVOT] in dive file
2. **Main IMMEDIATELY:** scan affected decisions + Impact Ripple (inline, ~30 sec)
3. **SHOW user:** "Pivot affects: {list of affected decisions}"
4. **DISPATCH** agent (Opus by default, or user-specified model) IN BACKGROUND with scoped checks
5. **CONTINUE** discussion — don't block momentum
6. **When agent returns:** integrate findings at next checkpoint

### Pivot-Specific Checks
Beyond standard gate checks, pivots MUST run:
- **IMPLICIT_KNOWLEDGE:** What did the old approach know/solve implicitly? What implicit assumptions carried weight that the new approach must explicitly account for?
- **HALT_WATCH triage:** Classify all affected decisions:
  - HALT — in-flight work that will become wrong (stop and redirect)
  - WATCH — planned work not yet started (reassess before starting)
- **Blast radius mapping:** Use dependency graph to trace which decisions reference the pivoted ones, directly or transitively.
