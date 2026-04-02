# ✈️ Flight Deck v2 — Specification

### Multi-Agent Software Development & Validation Framework

> *Three stages. Multiple AI minds. One validated product.*

**Author:** Huan Su
**Version:** 2.0 (Draft)
**Date:** March 2026
**Lineage:** Evolves from Flight Deck v1. Incorporates orchestration patterns inspired by Paperclip (github.com/paperclipai/paperclip).

---

## What Flight Deck v2 Is

Flight Deck v2 is a file-based, zero-infrastructure framework for orchestrating multiple AI agents through three distinct stages of software development: **planning together**, **building together**, and **validating through real use**.

It requires no server, no database, and no special tooling — only shared folders, `fswatch`, and AI coding agents that can read and write files.

**Core Principle:** Software is only "done" when multiple AI minds have debated the plan, built to consensus, and an AI agent has used the running application like a real human and confirmed genuine usability.

---

## The Three Stages

```
┌─────────────────────────────────────────────────────────────────┐
│                        FLIGHT DECK v2                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  STAGE 1: FLIGHT PLANNING                                       │
│  ─────────────────────────                                      │
│  Multi-LLM deliberation on build files before code is written.  │
│  Different AI models from different providers debate, challenge, │
│  and refine the PRD, CLAUDE.md, state machine, and architecture │
│  until they reach consensus.                                    │
│                                                                 │
│  Agents:  Claude Code + Codex + Gemini CLI (or any combo)       │
│  Comms:   Shared discussion folder, fswatch-triggered           │
│  Input:   Draft PRD, CLAUDE.md, state machine, schemas          │
│  Output:  Refined files (v2+) + DELIBERATION_LOG.md             │
│  Gate:    All agents signal agreement, OR human director approves│
│                                                                 │
│  ═══════════════════ GOVERNANCE GATE ════════════════════════    │
│                                                                 │
│  STAGE 2: FLIGHT (Building)                                     │
│  ──────────────────────────                                     │
│  Coding agents execute the agreed plan. The lead agent (Claude  │
│  Code) drives the build using the Ralph Wiggum Loop. Supporting  │
│  agents (Codex, etc.) may co-build or review.                   │
│                                                                 │
│  Agents:  Claude Code (lead) + Codex (co-builder/reviewer)      │
│  Comms:   Same repo, branch strategy, code review via files     │
│  Input:   Refined build files from Stage 1                      │
│  Output:  Running application + "Ready for test" signal         │
│  Gate:    Code compiles/runs, lead agent signals readiness       │
│                                                                 │
│  ═══════════════════ GOVERNANCE GATE ════════════════════════    │
│                                                                 │
│  STAGE 3: TEST FLIGHT                                           │
│  ────────────────────                                           │
│  Cowork Opus — pure computer use testing. Tests the running     │
│  application as different personas with different knowledge      │
│  levels. Reports results. Does NOT make decisions, does NOT      │
│  coordinate. Only tests.                                        │
│                                                                 │
│  Agent:   Cowork Opus (computer use, nothing else)              │
│  Comms:   Test Pilot Loop shared folder, fswatch-triggered      │
│  Input:   "Ready for test" signal + running application         │
│  Output:  Test results → FLIGHT_PLAN.md feedback                │
│  Gate:    All persona tiers pass, OR human director accepts      │
│                                                                 │
│  ═══════════════════ GOVERNANCE GATE ════════════════════════    │
│                                                                 │
│  DONE — or loop back to Stage 2 (fix) or Stage 1 (rethink)     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Agent Roles (Fixed, Not Hierarchical)

Flight Deck v2 uses a flat structure with clear, non-overlapping roles. There is no org chart or reporting hierarchy. Each agent has exactly one job.

| Role | Agent | Job | What It Does NOT Do |
|------|-------|-----|---------------------|
| **Planning Voices** | Claude Code, Codex, Gemini CLI | Debate and refine build files during Flight Planning | Write production code during Stage 1 |
| **Lead Builder** | Claude Code | Drive the build, coordinate with co-builders, signal readiness for testing | Test the app through computer use |
| **Co-Builder** | Codex (or other coding agent) | Co-build or review code during the Flight stage | Make architectural decisions unilaterally |
| **Test Pilot** | Cowork Opus | Test the running app through computer use as different personas | Make build decisions, coordinate agents, or fix bugs |
| **Human Director** | You | Override any governance gate, review logs, set budgets, rewind via Flight Recorder | Write code (unless you want to) |

---

## Stage 1: Flight Planning (Multi-LLM Deliberation)

### Purpose

Pressure-test the blueprint before construction starts. Multiple AI models from different providers — with different training data, different strengths, different blind spots — debate the approach until the build files are tight.

### What Gets Refined

Any file that determines downstream build quality:

- **PRD (Product Requirements Document)** — what are we building and why
- **CLAUDE.md** — coding agent behavior, constraints, conventions
- **State machine** — app flow logic, screen transitions, edge cases
- **API contracts, data models, schemas** — technical architecture
- **Any other key build file** the human director designates

### How It Works

```
FLIGHT_PLANNING/
├── INBOX/                    ← Human director drops draft files here
│   ├── PRD_draft.md
│   ├── CLAUDE_draft.md
│   └── STATE_MACHINE_draft.md
│
├── DISCUSSION/               ← Agents read/write here, fswatch triggers
│   ├── round_001_claude.md   ← Claude Code's analysis
│   ├── round_001_codex.md    ← Codex's response
│   ├── round_001_gemini.md   ← Gemini's response
│   ├── round_002_claude.md   ← Claude Code responds to both
│   ├── ...                   ← Continues until consensus
│   └── AGREEMENT.md          ← All agents append confirmation
│
├── OUTPUT/                   ← Refined files, ready for Stage 2
│   ├── PRD_v2.md
│   ├── CLAUDE_v2.md
│   └── STATE_MACHINE_v2.md
│
└── DELIBERATION_LOG.md       ← Immutable, append-only full record
```

### fswatch Protocol for Flight Planning

Each agent watches `FLIGHT_PLANNING/DISCUSSION/` with `fswatch`. When a new file appears:

1. Agent reads ALL existing discussion files (not just the new one)
2. Agent considers the full context: original drafts + all prior rounds
3. Agent writes its response as a new file: `round_NNN_agentname.md`
4. Response must include: what it agrees with, what it challenges, what it proposes, and whether it's ready to agree

### Turn-Taking Convention

To prevent simultaneous writes:
- Agent creates a lock file: `.thinking_agentname` before composing
- Other agents wait if any `.thinking_*` file exists
- Agent removes the lock file after writing its response
- If a lock file is older than 5 minutes, it's considered stale and can be ignored

### Agreement Protocol

An agent signals agreement by appending to `AGREEMENT.md`:

```markdown
## [Agent Name] — [Timestamp]
Status: AGREE
Conditions: None (or: "Agree contingent on X being addressed in build")
```

When all participating agents have appended AGREE status, Stage 1 is complete. The refined files in `OUTPUT/` become the build instructions for Stage 2.

### Deliberation Budget

**Budget cap per agent for Stage 1.** If an agent hits its Flight Planning budget, it must write a final summary of its position and exit deliberation. Remaining agents continue. This prevents infinite debates.

Default recommendation: cap Flight Planning at 20% of total project budget.

---

## Stage 2: Flight (Building)

### Purpose

Execute the agreed plan from Stage 1. The lead builder (Claude Code) drives the build using the Ralph Wiggum Loop. Co-builders may work on separate features or review code.

### Communication During Building

During the build phase, coding agents communicate through:
- The shared repository (PRs, branches, code review via comments in files)
- The `FLIGHT_PLANNING/DISCUSSION/` folder can be re-used if a build-time question arises that needs multi-agent deliberation — this triggers a mini Flight Planning round scoped to the specific question

### Signaling "Ready for Test"

When the lead builder determines a feature or full build is ready for testing, it writes to the Test Pilot Loop shared folder:

```markdown
## Ready for Test — [Timestamp]

Feature: Photo upload via drag-and-drop
App location: http://localhost:3000
What to test: Upload 1-10 photos, verify they appear in grid
Known limitations: Only JPEG supported in this build
Goal ancestry: Mission > PhotoSortVision MVP > Photo Import > Upload Flow
```

The **goal ancestry** line traces the test back to the mission, so the test pilot always knows *why* this matters.

---

## Stage 3: Test Flight

### Purpose

Validate the running application through actual use. Cowork Opus is a **pure tester** — it uses computer use to interact with the app as different personas with different knowledge levels, reports results, and nothing else.

### Test Pilot Protocol

The existing Test Pilot Loop protocol applies here unchanged:
- Four personas: Alex (Power User), Jordan (Everyday User), Pat (Struggling User), Sam (Accessibility User)
- Three knowledge tiers: Cold (knows nothing), Guided (has user manual), Insider (knows PRD)
- Tier Gap Ratio: Cold user steps ÷ Insider steps = intuitiveness metric
- Screen-by-screen systematic inspection: screenshot → analyze → hypothesize → act → evaluate

### What Changed from v1

In v1, Cowork Opus was the brain, the patrol monitor, AND the test pilot. In v2:
- Cowork Opus is ONLY the test pilot
- Claude Code (the lead builder) decides WHEN to trigger testing
- Cowork does not patrol — it responds to test signals via fswatch
- Cowork does not make architectural decisions or coordinate agents

### Persistent Test State

Opus maintains awareness across test cycles via `TEST_STATE.md`:

```markdown
## Test State — Last Updated [Timestamp]

### Current Cycle
- Feature under test: Photo upload
- Persona: Cold User (Pat)
- Progress: Step 3 of 7 — FAILED at drag-and-drop
- Blocker: Drop zone not visually indicated

### Previous Cycles
- Cycle 1: Cold User upload — FAILED step 3 (drag-and-drop)
- Cycle 0: Initial screen analysis — PASSED (app loads correctly)
```

On each wake-up, Opus reads TEST_STATE.md first to understand where things stand, avoiding redundant re-analysis.

### Test Results and Feedback Loop

After testing, Opus writes structured results to `FLIGHT_PLAN.md`:

```markdown
## Test Results — [Timestamp]

Persona: Pat (Cold User)
Tier: Cold (no prior knowledge)
Feature: Photo upload

### Findings
- FAIL: No visual indicator for drag-and-drop zone
- FAIL: "Import" button label unclear — tested user expected "Upload"
- PASS: File picker dialog works correctly
- PASS: Progress indicator shows during upload

### Tier Gap Ratio
- Cold user: 9 steps to upload successfully
- Insider: 3 steps
- Ratio: 3.0 (target: < 2.0)

### Recommendation
Block — drag-and-drop UX needs redesign before Guided and Insider testing
```

Claude Code reads this, fixes the issues, and signals "Ready for retest."

---

## Governance Gates

Governance gates control stage transitions. They can be **automatic** (for overnight runs) or **human-approved** (when the director wants control).

### Gate Definitions

| Gate | From → To | Automatic Condition | Human Override |
|------|-----------|-------------------|----------------|
| **Planning Complete** | Flight Planning → Flight | All agents AGREE in AGREEMENT.md | Director approves plan before agents finish |
| **Build Ready** | Flight → Test Flight | Code compiles/runs + lead signals "Ready for test" | Director can trigger test at any point |
| **Test Passed** | Test Flight → Done | All required persona/tier combos pass | Director accepts with known issues |
| **Test Failed** | Test Flight → Flight | Any persona/tier combo fails | Director can override to "Done" |
| **Rethink** | Test Flight → Flight Planning | Fundamental design issue discovered in testing | Director decides to go back to planning |
| **Rollback** | Any stage → Previous checkpoint | Regression detected or agent requests rollback | Director rewinds via Flight Recorder |

### Overnight Mode vs. Supervised Mode

**Overnight (autonomous):** All gates are automatic. Agents cycle through stages without human intervention. Budget caps and the Flight Recorder provide safety nets.

**Supervised:** Human director must approve each gate transition. Agents pause and wait for approval at each gate.

**Hybrid (recommended):** Automatic gates for Flight → Test Flight → Flight (the build-test-fix loop). Human approval required for Planning Complete and Done.

---

## Budget Circuit Breakers

Every agent has a spending cap. This prevents runaway costs during overnight autonomous runs.

### Budget Structure

```markdown
## BUDGET.md

### Project Budget
Total project cap: $50.00

### Per-Stage Budgets
Flight Planning:  $10.00 (20% of total)
Flight (Building): $25.00 (50% of total)
Test Flight:       $15.00 (30% of total)

### Per-Agent Budgets (within stage budgets)
Claude Code (planning):   $4.00
Codex (planning):         $3.00
Gemini CLI (planning):    $3.00
Claude Code (building):   $20.00
Codex (building):         $5.00
Cowork Opus (testing):    $15.00

### Thresholds
Warning:  80% of budget consumed → agent logs warning, continues
Hard cap: 100% of budget consumed → agent stops, writes exit summary

### Actual Spend (append-only)
- [02:14] Claude Code planning: $0.45 (11% of planning budget)
- [02:31] Codex planning: $0.38 (13% of planning budget)
- ...
```

### What Happens at Budget Cap

When an agent hits 100% of its budget:

1. Agent stops immediately
2. Agent writes an exit summary: what it was doing, where it stopped, what remains
3. Agent updates FLIGHT_LOG.md with the budget halt
4. Other agents continue if they have budget remaining
5. Human director can override: increase budget and resume

---

## Flight Recorder (Time Machine)

> **🕐 THIS IS A TIME MACHINE.** Every checkpoint is a frozen snapshot of the entire project state. You can rewind to any checkpoint and resume from that point, undoing everything that happened after it.

### Purpose

The Flight Recorder captures the project state at meaningful moments — stage transitions, successful tests, pre-regression snapshots — so that the human director or the lead agent can **rewind time** when things go wrong.

### How It Works

```
FLIGHT_RECORDER/
├── README.md                 ← Explains this is a Time Machine
│
├── checkpoint_001/
│   ├── SNAPSHOT.md            ← What happened, who was involved, why this checkpoint exists
│   ├── timestamp.txt          ← ISO 8601 timestamp
│   ├── stage.txt              ← Which stage was active (planning / flight / test_flight)
│   ├── git_commit.txt         ← Git commit hash at this moment
│   ├── files/                 ← Frozen copies of all key build files
│   │   ├── PRD.md
│   │   ├── CLAUDE.md
│   │   ├── STATE_MACHINE.md
│   │   └── ...
│   ├── test_state.md          ← TEST_STATE.md at this moment (if applicable)
│   └── budget_state.md        ← Budget consumption at this moment
│
├── checkpoint_002/
│   └── ...
│
└── TIMELINE.md                ← Human-readable timeline of all checkpoints
```

### FLIGHT_RECORDER/README.md

```markdown
# 🕐 Flight Recorder — Time Machine

This folder contains frozen snapshots of the project at key moments.

## HOW TO USE (for AI agents and humans)

**To rewind:** Write a file called ROLLBACK.md in this folder with:

    target: checkpoint_NNN
    reason: "Brief explanation of why we're rewinding"

The lead agent (Claude Code) will:
1. Read the target checkpoint
2. Restore all build files from checkpoint_NNN/files/
3. Git checkout to the commit in checkpoint_NNN/git_commit.txt
4. Resume from the stage recorded in checkpoint_NNN/stage.txt
5. Log the rollback in FLIGHT_LOG.md

**To browse:** Read TIMELINE.md for a human-readable summary of all checkpoints.

## RULES
- Checkpoints are IMMUTABLE — never edit or delete a checkpoint
- Only the lead agent or human director can trigger a rollback
- Every rollback creates a NEW checkpoint (the rollback itself is recorded)
- The Flight Recorder is append-only — rewinding does NOT delete future checkpoints
```

### When Checkpoints Are Created

| Trigger | Checkpoint Name Pattern |
|---------|------------------------|
| Stage 1 complete (agreement reached) | `checkpoint_NNN_planning_complete` |
| Before each test cycle begins | `checkpoint_NNN_pre_test` |
| After a test cycle passes | `checkpoint_NNN_test_passed` |
| Before a major code change | `checkpoint_NNN_pre_change` |
| After a rollback is executed | `checkpoint_NNN_rollback_from_MMM` |
| Human director manually triggers | `checkpoint_NNN_manual` |

### TIMELINE.md Example

```markdown
# Flight Recorder Timeline

| # | Time | Stage | Event | Notes |
|---|------|-------|-------|-------|
| 001 | 02:30 | Planning | Planning complete | 3 agents agreed after 4 rounds |
| 002 | 03:15 | Flight | Pre-test snapshot | Upload feature built |
| 003 | 03:28 | Test Flight | Cold user FAILED | Drag-and-drop not discoverable |
| 004 | 03:45 | Flight | Pre-test snapshot | Drag-and-drop fix applied |
| 005 | 03:58 | Test Flight | Cold user PASSED | Upload flow validated |
| 006 | 04:20 | Flight | Pre-test snapshot | Sort feature built |
| 007 | 04:35 | Test Flight | Cold user FAILED | Sort crashed on 10K+ photos |
| 008 | 04:50 | Flight | ROLLBACK to 005 | Sort fix caused upload regression |
```

---

## Immutable Audit Log (FLIGHT_LOG.md)

Every significant action across all three stages is appended to `FLIGHT_LOG.md`. Agents **never edit or delete** existing entries.

### Format

```markdown
# Flight Log — [Project Name]

## Entries (append-only)

[02:00] STAGE_CHANGE | → Flight Planning
[02:01] AGENT_JOIN | Claude Code enters Flight Planning
[02:01] AGENT_JOIN | Codex enters Flight Planning
[02:02] AGENT_JOIN | Gemini CLI enters Flight Planning
[02:05] DELIBERATION | Claude Code: round_001 — identified 3 PRD gaps
[02:12] DELIBERATION | Codex: round_001 — agreed on 2 gaps, challenged 1
[02:18] DELIBERATION | Gemini: round_001 — proposed state machine additions
[02:25] DELIBERATION | Claude Code: round_002 — revised proposal
[02:30] AGREEMENT | All agents agree — Planning complete
[02:30] CHECKPOINT | checkpoint_001_planning_complete
[02:30] GATE_PASS | Planning Complete → Flight (automatic)
[02:31] STAGE_CHANGE | → Flight
[02:31] BUILD_START | Claude Code begins upload feature
[03:15] BUILD_SIGNAL | Claude Code: "Ready for test" — upload feature
[03:15] CHECKPOINT | checkpoint_002_pre_test
[03:15] GATE_PASS | Build Ready → Test Flight (automatic)
[03:16] STAGE_CHANGE | → Test Flight
[03:16] TEST_START | Cowork Opus: Cold user test — upload feature
[03:28] TEST_RESULT | FAIL — drag-and-drop not discoverable (Tier Gap: 3.0)
[03:28] GATE_PASS | Test Failed → Flight (automatic)
[03:28] STAGE_CHANGE | → Flight
[03:29] BUILD_START | Claude Code begins drag-and-drop fix
[03:45] BUDGET_WARNING | Codex at 80% of planning budget
[04:50] ROLLBACK | → checkpoint_005 (reason: sort fix caused upload regression)
[04:50] CHECKPOINT | checkpoint_008_rollback_from_005
```

---

## Goal Ancestry Chain

Every task, build action, and test carries a "because" trail linking it to the project mission. This ensures agents always know WHY they're doing something, not just WHAT.

### Structure

```
Mission: "Build PhotoSortVision MVP — AI-powered photo organizer"
  └─ Objective: "Photo Import Flow"
      ├─ Build Task: "Implement drag-and-drop upload"
      │   └─ Test Task: "Cold user — can they upload without instructions?"
      └─ Build Task: "Implement file picker fallback"
          └─ Test Task: "Accessibility user — can they upload via keyboard?"
  └─ Objective: "Photo Organization"
      ├─ Build Task: "Implement AI-powered auto-sort"
      └─ Build Task: "Implement manual folder creation"
```

### How Agents Use It

When Claude Code signals "Ready for test," it includes the goal ancestry:

```
Goal ancestry: Mission > Photo Import > Drag-and-drop Upload
```

When Opus reports test failure, it includes the same chain:

```
FAIL: Drag-and-drop not discoverable
Impact: Blocks Photo Import objective → blocks MVP mission
```

When the human director reads the morning debrief, they instantly see which objectives are validated and which are blocked — not just which buttons passed or failed.

### Goal Ancestry File

```markdown
## GOAL_ANCESTRY.md

Mission: Build PhotoSortVision MVP

### Objectives
1. Photo Import Flow
   - 1.1 Drag-and-drop upload [status: testing]
   - 1.2 File picker fallback [status: planned]
   - 1.3 Batch import from folder [status: planned]

2. Photo Organization
   - 2.1 AI auto-sort by content [status: not started]
   - 2.2 Manual folder creation [status: not started]

3. Photo Viewing
   - 3.1 Grid view [status: not started]
   - 3.2 Full-screen viewer [status: not started]
```

---

## Folder Structure

```
project-root/
├── FLIGHT_DECK/
│   ├── FLIGHT_PLANNING/
│   │   ├── INBOX/                    ← Draft files from human director
│   │   ├── DISCUSSION/               ← Multi-LLM deliberation (fswatch)
│   │   ├── OUTPUT/                   ← Refined files for Stage 2
│   │   └── DELIBERATION_LOG.md       ← Append-only deliberation record
│   │
│   ├── TEST_PILOT/
│   │   ├── FLIGHT_PLAN.md            ← Test signals + results (fswatch)
│   │   └── TEST_STATE.md             ← Persistent test state across cycles
│   │
│   ├── FLIGHT_RECORDER/              ← 🕐 TIME MACHINE — rewind to any checkpoint
│   │   ├── README.md                 ← Explains Time Machine for agents & humans
│   │   ├── TIMELINE.md              ← Human-readable checkpoint timeline
│   │   ├── checkpoint_001_.../
│   │   ├── checkpoint_002_.../
│   │   └── ...
│   │
│   ├── FLIGHT_LOG.md                 ← Immutable append-only audit trail
│   ├── FLIGHT_DEBRIEF.md             ← Final summary after all testing
│   ├── BUDGET.md                     ← Per-agent spending caps + actuals
│   ├── GATES.md                      ← Stage transition rules
│   ├── GOAL_ANCESTRY.md              ← Mission → Objective → Task tracing
│   │
│   └── PERSONAS/                     ← Test personas (unchanged from v1)
│       ├── alex_power_user.md
│       ├── jordan_everyday_user.md
│       ├── pat_struggling_user.md
│       └── sam_accessibility_user.md
│
├── PRD.md                            ← Product requirements (refined by Stage 1)
├── CLAUDE.md                         ← Coding agent instructions (refined by Stage 1)
├── STATE_MACHINE.md                  ← App flow logic (refined by Stage 1)
└── src/                              ← Application source code
```

---

## Getting Started

### Prerequisites

- Two or more AI coding agents installed (Claude Code + Codex recommended minimum)
- Cowork with Opus for computer use testing
- `fswatch` installed (`brew install fswatch` on macOS)
- A project with draft PRD and CLAUDE.md

### Quick Start

**1. Set up the folder structure**

```bash
mkdir -p FLIGHT_DECK/{FLIGHT_PLANNING/{INBOX,DISCUSSION,OUTPUT},TEST_PILOT,FLIGHT_RECORDER,PERSONAS}
```

**2. Drop your draft files into INBOX**

```bash
cp PRD_draft.md FLIGHT_DECK/FLIGHT_PLANNING/INBOX/
cp CLAUDE_draft.md FLIGHT_DECK/FLIGHT_PLANNING/INBOX/
```

**3. Start agent watchers**

Terminal 1 — Claude Code:
```bash
fswatch -o FLIGHT_DECK/FLIGHT_PLANNING/DISCUSSION/ | while read; do
  # Claude Code reads new files and responds
  claude "Read all files in FLIGHT_DECK/FLIGHT_PLANNING/DISCUSSION/ and the drafts in INBOX/. Write your analysis as the next round file."
done
```

Terminal 2 — Codex:
```bash
fswatch -o FLIGHT_DECK/FLIGHT_PLANNING/DISCUSSION/ | while read; do
  # Codex reads and responds
  codex "Read all files in FLIGHT_DECK/FLIGHT_PLANNING/DISCUSSION/ and the drafts in INBOX/. Write your response as the next round file."
done
```

Terminal 3 — Cowork Opus (test pilot, waits for test signals):
```bash
fswatch -o FLIGHT_DECK/TEST_PILOT/FLIGHT_PLAN.md | while read; do
  # Cowork reads test signal and begins testing
done
```

**4. Configure budgets in BUDGET.md**

**5. Set governance gates in GATES.md** (automatic, supervised, or hybrid)

**6. Go to sleep.** Read FLIGHT_LOG.md and FLIGHT_RECORDER/TIMELINE.md in the morning.

---

## Key Differences from v1

| Aspect | Flight Deck v1 | Flight Deck v2 |
|--------|---------------|----------------|
| Brain / orchestrator | Cowork Opus | Claude Code |
| Pre-build deliberation | None | Multi-LLM Flight Planning stage |
| Cowork role | Brain + patrol + tester | Pure tester only |
| Agent communication | FLIGHT_PLAN.md only | Two channels (discussion + test signals) |
| Cost control | None | Budget circuit breakers per agent |
| Rollback capability | None | Flight Recorder (Time Machine) |
| Stage transitions | Implicit | Governance gates (automatic or human) |
| Task traceability | None | Goal ancestry chain |
| Audit trail | FLIGHT_LOG (basic) | FLIGHT_LOG (immutable, structured) |
| Test continuity | Cold restart each cycle | Persistent TEST_STATE.md |
| Models | Opus only | Multi-model, multi-provider |

---

## Design Decisions and Rationale

**Why file-based, not server-based?**
Paperclip requires Node.js + Postgres. Flight Deck requires only files and fswatch. Any coding agent that can read and write files can participate. Zero infrastructure means zero setup friction and zero maintenance burden.

**Why Claude Code as brain instead of Cowork?**
Claude Code is stable, reliable, CLI-based, and handles complex coordination well. Cowork's computer use capability is valuable but still buggy for orchestration tasks. By limiting Cowork to only computer use testing, we use each tool for what it's best at.

**Why multi-LLM deliberation?**
Different AI models have different training data, different reasoning patterns, and different blind spots. A plan reviewed by Claude, Codex, and Gemini is more robust than one reviewed by any single model. This is the automated version of what the human director already does — consulting multiple AI systems and integrating their feedback.

**Why budget caps?**
Overnight autonomous runs without cost limits are gambling. Budget circuit breakers are the single most important safety feature for unattended operation.

**Why a Time Machine?**
Software development is nonlinear. Fixes create regressions. New features break old ones. The ability to rewind to a known-good state and try a different approach is essential for overnight autonomous operation where no human is watching.

---

## Acknowledgments

Flight Deck v2 incorporates orchestration patterns inspired by [Paperclip](https://github.com/paperclipai/paperclip), an open-source multi-agent orchestration platform. Specifically: goal ancestry chains, budget circuit breakers, persistent agent state, governance gates, and immutable audit logs. Flight Deck adapts these patterns for a file-based, zero-infrastructure context.

The Test Pilot Loop, persona system, knowledge tiers, and Tier Gap Ratio are original to this framework.
