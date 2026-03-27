<!-- Full BPR process. Extends bpr-character.md (behavioral core). -->
<!-- Loaded by SKILL.md router when NOT in lite mode. -->

# BPR Full Mode — Process & File Management

> **Extends:** `bpr-character.md` (behavioral core — critique tone, anti-drift, response classification)
> **Required reading:** `bpr-gate.md` (gate system) + `bpr-anti-drift.md` (full anti-drift with periodic/boundary tiers)

# File Structure

```
.claude/reviews/big-picture-review/
└── {YYYY-MM-DD}T{HH-MM}-{topic-slug}/
    ├── master.md       # Index + short summary of each depth's decisions (<100 lines)
    ├── dive-0-1.md     # Depth 0, part 1 (<300 lines each)
    ├── dive-0-2.md     # Depth 0, part 2 (if split needed)
    ├── dive-1-1.md     # Depth 1, part 1
    └── ...
```

**Directory bootstrapping:** On first write, create the directory via Bash:
```bash
mkdir -p .claude/reviews/big-picture-review/{YYYY-MM-DD}T{HH-MM}-{topic-slug}
```
NEVER write review files to `.claude/plans/` or any other directory.

**Topic-slug generation:** lowercase, replace spaces/underscores with hyphens, strip non-alphanumeric except hyphens. For non-Latin topics: use English translation or transliteration (e.g., "архитектура сайдбара" → "sidebar-architecture"). Max 40 chars, truncate at word boundary.

**master.md template:**
```markdown
# Review: {topic}
Started: {date} | Current depth: {N} | Status: in-progress
Vision source: {path or "D0-conversation" or "none"}
Style: {preference}

## Vision Baseline
{5-10 key goals/principles extracted from vision source, or from D0 conversation}

## Depth 0: {title}
- Subtopics covered: {list}
- Key decisions: {1-2 lines each}
- Open questions: {list}
- Dive files: dive-0-1.md [, dive-0-2.md, ...]
- Validation: {done|pending|recycle} ({date}) | Result: {GO|RECYCLE|KILL} | Findings: {summary}

## Depth 1: ...
(added incrementally per depth)

## Final Summary (added when Done)
- Risk register (unresolved Medium/High issues)
- Open questions
- Assessment
```

**Compression rule:** If master.md exceeds 100 lines at checkpoint: compress older depths to 2 lines each. Mention once: "Compressing older summaries, full details in dive files." Master = index, not journal.

**Dive file template** (`dive-{D}-{N}.md`):
```markdown
# Depth {D}, Part {N}: {depth preamble title}
Review: {topic} | Date: {date}

## Subtopics
1. {name} — {status: covered/skipped/incomplete}

## Critiques

### {Subtopic 1}
**Pre-mortem:** {2-3 sentences, concrete failure scenario}
**Issues:**
- [{severity}] [{epistemic tag}] {problem + mechanism + impact}
**Strengths:** {optional, if <2 issues: "No major issues. Solid because: {reason}."}
**Approaches:** {integrated approaches covering ALL issues, with trade-offs}
**User decision:** {what user chose + rationale}
**Affected files:**
- INVALIDATES: {file} (reason)
- UPDATE NEEDED: {file:lines} (reason)
- SUPERSEDES: {dive decision reference}

### {Subtopic 2}
...

## Open Questions
## Next Depth Suggestions
```

**Split rule:** If dive file exceeds 300 lines, split: `dive-{D}-{N+1}.md`. Master.md tracks parts.

**Carry-forward (mandatory on split):** New dive part MUST open with:
```
## Carry-Forward from dive-{D}-{N}.md
- Decisions D{first}–D{last} carried forward (brief summary: {list})
- Open constraints: {list}
- Superseded: {list}
- AWAITING APPROVAL: {list}
- Last 3-5 decisions as read-only context
```
Use absolute references only (D0-decision-7, never "the previous decision").

- **master.md**: compact index, grows incrementally, stays <100 lines even at depth 5
- **dive-{D}-{N}.md**: rich format per template above
- Previous depth plans NOT re-read in full — master.md contains their summary
- **Recovery:** if you lose context, read master.md + current dive file before continuing

**Post-write validation (INTERNAL):** After writing any file, verify via Read: file exists, non-empty, contains expected header (`# Review:` for master.md, `# Depth {D}` for dive files).

# Workflow

## START

```
1. Glob for .claude/reviews/big-picture-review/*-{topic-slug}/master.md
   If multiple matches: pick most recent by date prefix
2. If found:
   - Read master.md, determine last completed depth
   - Read "Vision source:" field — if present, vision is already configured
   - If "Vision source:" field is missing (old review): ask once "Do you have a project vision/goals document? [path / Skip]" and write to master.md header
   - Re-ask explanation preference (even if stored in master.md as fallback)
   - Save user's style choice to master.md header (e.g., `Style: short-and-clear`) for fallback on future resumes
   - Resume protocol:
     a. External change scan: files in review dir changed since last save?
     b. Staleness classification: time-only / change-triggered / hard miss / assumption-violated
     c. Show SBAR summary from last pause (if exists)
     d. Synthesis confirmation: user acknowledges before continuing (I-PASS)
     e. Re-ask style preference
     f. If master.md shows `Validation: pending` → run gate first
   - AskUser: "Last completed: depth {N}. Continue at depth {N+1}? [Yes / Different depth / New review]"
3. If not found:
   - mkdir -p .claude/reviews/big-picture-review/{date}-{topic-slug}
   - Glob for other master.md in .claude/reviews/big-picture-review/
     → If any has "Vision source:" field (not "none", not "D0-conversation") →
       AskUser: "Previous review used {path}. Use same?"
       - "Yes, same document" — reuse path, re-extract baseline
       - "Different document" — ask for new path
       - "No document, build from conversation" — defer to D0 extraction
       - "Skip vision tracking" — VISION_DRIFT disabled
     → If none found → defer vision question to D0 (step 3c below)
   - Create fresh master.md from template
   - Start at D0
3c. Vision collection (during D0, after user explains their topic):
   AskUser (header: "Vision"):
     "Do you have a project vision/goals document? (PRD, design doc, README with goals)"
     - "Yes" — I'll provide a path to the vision document
     - "No document, build from conversation" — extract vision from our D0 dialogue
     - "Skip vision tracking" — VISION_DRIFT will be disabled
   → Yes → ask for path → read, extract 5-10 key points, write to master.md "Vision Baseline" section
   → No document → write "Vision source: D0-conversation (pending)", after D0 dialogue extract key goals/principles, show user for confirmation/edit, update to "Vision source: D0-conversation" + write Vision Baseline
   → Skip → write "Vision source: none", leave Vision Baseline empty, VISION_DRIFT skipped at gates
4. Detect user-driven mode:
   If user's topic argument contains numbered list / multiple items / explicit question list:
   → Switch to participant-driven mode:
     - User's items = subtopic map (not AI-generated)
     - Claude's role shifts to COVERAGE AUDITOR:
       (a) Gap detection: scan space NOT covered by user's list
       (b) Cross-referencing: find connections user doesn't see
       (c) Re-prioritization: objective sort by impact/risk
       (d) Complementary perspective: structural/systemic angle
       (e) Questions about questions: "why these items?"
     - All other BPR mechanics (critique, anti-drift, gates) unchanged
```

After start, ask explanation preference:
AskUser (header: "Style"):
  "How much explanation do you want?"
  - "More explanations" — explain concepts and mechanisms
  - "Short and clear" — minimal, assume I understand
  - "Explain when needed" — adaptive default

## D0: FREE DIALOGUE

D0 is conversational, not transactional. Minimal AskUser.

```
1. Ask user to explain their intent (what, why, for whom) — TEXT, not buttons
1b. Research relevant code/docs as needed (Read/Grep/Glob or Explore agents).
    Web research available without gate (up to 3/depth, text notification: "Looking up {topic}...").
2. Restate understanding: "So the core problem is X and you want Y"
3. Show 2-3 professional/philosophical lenses on the same problem
   "A frontend architect sees this as... A UX designer sees this as..."
4. Let user respond freely. Follow their thread.
5. Use subtopics as WAYPOINTS, not a checklist:
   - Identify subtopics internally as a map
   - When conversation naturally covers a subtopic — note it
   - When user seems lost: "We've explored X and Y. Haven't touched Z — relevant?"
6. For each topic naturally reached:
   - OUTPUT critique as text, adapting Critique Format to dialogue style:
     Pre-mortem + issues (tags internal) + approaches with trade-offs.
     Not a formal block — woven into conversation naturally.
   - If user engages → continue dialogue (grill is built into the conversation)
   - If user seems decided → briefly note the approach chosen
   - Alternative approaches (item 5) apply here too — D0 is where "works but
     there's another way" is most valuable for expanding user's perspective
7. Checkpoint every 5+ rounds: SAVE to dive-0-1.md (don't interrupt, just save)
8. When D0 feels complete (user's intent is clear, blind spots explored):
   INTERNAL self-check: "What important aspect of this topic have we NOT examined?"
   If substantive finding — raise naturally in dialogue before suggesting transition.
   If only Minor items — proceed to transition.
   (Diminishing returns guard: 2 consecutive self-checks yield only Minor → D0 is done.)
   AskUser: "D0 covered {summary}. Go deeper? [D1 / Done / Continue D0]"
   (No "Jump to depth" option — after D0 there's nowhere to jump yet.
   Jump appears in DEPTH TRANSITION after D1+.)
```

## D1+: SEMI-STRUCTURED → STRUCTURED

As depth increases, interaction becomes more structured:

```
D1: Text dialogue (like D0, but more structured)
  - Critique as text
  - User responds freely (choice, idea, question, new direction)
  - After user indicates preferred approach, challenge with 1-2 questions before recording.
    Skip only if already fully critiqued in same exchange.
  - Navigation through text: "We've covered X. Y and Z remain — want to explore those, or done with this depth?"

D2+: Text dialogue, increasingly concrete
  - Less philosophical discussion, more concrete trade-offs
  - [Fact] issues get direct language. [Judgment] issues get balanced trade-offs.
  - After user indicates preferred approach, always challenge before recording.
```

## NAVIGATION (INTERNAL — applies to ALL depths)

Subtopics are a MAP, not RAILS:
- Claude tracks covered/uncovered subtopics internally
- When user naturally covers a subtopic → mark it
- Periodically: "We've covered X. {Y and Z} remain — want to explore those?"
- User can ignore the map and go wherever they want
- Map helps RETURN to course, not dictate it
- D0: navigation through natural dialogue ("Haven't touched Z — relevant?")
- D1+: navigation through text dialogue ("We've covered X. Y and Z remain — want to explore those, or done with this depth?")
- When moving to next subtopic → SAVE current subtopic to dive file regardless of round count (even if <5 exchanges)

## SAVE CHECKPOINTS

Every 5+ exchanges:
- Save to current depth's dive file: `dive-{D}-{N}.md` (e.g., dive-0-1.md, dive-1-1.md)
- Read + append + Write pattern
- Do NOT interrupt the user: "Saving progress..." is sufficient
- This is about CONTEXT PRESERVATION, not pacing
- If dive file > 300 lines → split: increment N (e.g., dive-0-2.md)
- Before saving: check for AWAITING APPROVAL decisions. List at next user interaction.
- When moving to next subtopic: save current subtopic regardless of round count.
- Open Questions Revisit: re-read Open Questions from current dive with accumulated context.
  If resolved → show user: "Earlier I asked X. Given our discussion — here's an answer: {insight}"
  Binary choices: try to find integrative 3rd option combining strengths of both.

## DEPTH TRANSITION

```
INTERNAL self-check before suggesting transition:
  D0: "What important aspect have we NOT examined?"
  D1: "Do chosen approaches interact? Conflicts between decisions?"
  D2+: "Edge cases at intersection of discussed mechanisms?"
If substantive → raise in dialogue. If only Minor → proceed.
Diminishing returns guard: 2 consecutive self-checks yield only Minor → depth is done.

Open Questions Revisit: re-read Open Questions from current dive with accumulated context.
  If resolved → show user: "Earlier I asked X. Given our discussion — here's an answer: {insight}"
  Binary choices: try to find integrative 3rd option combining strengths of both.

Before presenting transition AskUser:
  MANDATORY: Run Transition Gate (see bpr-gate.md).
  Gate result determines options:
  - Go → present D+1 option
  - Recycle → return to current depth, reopen subtopics
  - Kill → final summary, close review
  Only if Go: present full AskUser with D+1/Done/Continue/Jump options.

AskUser: "Depth {N} covered {summary}. What next?"
- "Depth {N+1}: {suggested focus}"
- "Done — write final summary"
- "Continue this depth"
- "Jump to specific depth"
```

# Question Contexts (INTERNAL)

| Context | When | Format |
|---------|------|--------|
| **D0 dialogue** | Throughout D0 | Text responses, minimal AskUser |
| **Post-critique (D1+)** | After critique output | Text: user responds to critique freely |
| **Post-approach challenge** | After user indicates preferred approach | Text, 1-2 challenge questions |
| **Checkpoint** | Every 5+ rounds | Brief "saving" note |
| **Depth transition** | End of depth | AskUser: next/done/continue/jump |

## Response Classification: Redirect (Full Mode Addition)

> In lite mode, redirects are noted mentally and raised at wrap-up.
> In full mode, redirects require file tracking:

7. **redirect** → user changes topic while decision pending
   - Mark current decision "AWAITING APPROVAL" in dive file
   - Follow user's new topic
   - At next checkpoint: "We didn't close: {list}. Confirm or revisit?"

## Dead Ends: Full Mode Additions

| # | Dead end | Resolution |
|---|----------|------------|
| 5 | [Fact] bad decision, user insists | [Fact] grill ALWAYS fires. Not to re-argue — to acknowledge: "Known risk. Logged in dive file." |
| 6 | Long D0, no orientation | After 10+ rounds: light map. "Covered {X}. Open: {Y}. Continue or transition?" |
| 7 | master.md > 100 lines | At checkpoint when > 100 lines: compress older depths (2 lines instead of 5) + mention once: "Compressing older summaries, full details in dive files." Master = index, not journal. |
| 8 | Revisit covered subtopic | Read dive file section → summarize decision → reopen for discussion. |

# DONE (Final Summary)

When user selects "Done" at depth transition:
1. Read master.md + all dive files
2. Write Final Summary into master.md:
   - Decisions overview (all depths, 1-2 lines each)
   - Open Questions (aggregated from all depths)
   - Risk register (unresolved Medium/High/Critical issues)
   - Assessment
3. Offer: "Write implementation plan? Recommend `writing-plans skill (if available)` with context from master.md"
4. If `.claude/handoff.md` exists, update it:
   - What was just done (review topic, depths covered)
   - What's next (open questions, recommended actions)
   - Relevant files (full paths to master.md, dive files)

# Early Exit (Pause Protocol)

If user says "stop"/"enough"/"pause" mid-depth:
1. Run Gate in PAUSE mode (see bpr-gate.md) — relaxed entrance, same checks
2. Save progress in SBAR format:
   - Situation: current state of review
   - Background: what led here
   - Assessment: Claude's evaluation + ISSUES_AT_PAUSE from gate
   - Recommendation: specific next action for resume
   + Assumption audit, failed approaches log, timestamp all state
3. Update master.md: "Depth N: paused (date) | ISSUES_AT_PAUSE: {count}"
4. Pause gate findings → ISSUES_AT_PAUSE in master.md
5. Critical findings don't block pause, but record prominently
