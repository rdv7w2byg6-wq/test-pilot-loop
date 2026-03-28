# Flight Plan — Dual-Patrol Coordination Document v0.2.0

> **Two agents. One shared file. No direct commands.**
> Both agents patrol this document independently. Neither controls the other.
> Communication happens exclusively through STATUS, NEXT_ACTION, and FINDINGS.

---

## 🏗️ ARCHITECTURE OVERVIEW

```
┌──────────────────────────────────────────────────┐
│              HUMAN DIRECTOR (You)                 │
│   Starts both agents. Final authority.            │
│   Intervenes on ESCALATE_TO_HUMAN.                │
└─────────────────────┬────────────────────────────┘
                      │ launches separately
          ┌───────────┴───────────┐
          ▼                       ▼
┌──────────────────┐   ┌──────────────────┐
│  CLAUDE CODE     │   │  COWORK          │
│  (Builder)       │   │  (Test Pilot)    │
│                  │   │                  │
│  Runs in Terminal│   │  Runs in Cowork  │
│  Builds code     │   │  Tests app via   │
│  Fixes bugs      │   │  computer use    │
│  Runs xcodebuild │   │  Finds UX bugs   │
│                  │   │                  │
│  Patrols this    │   │  Patrols this    │
│  file every      │   │  file every      │
│  10 minutes      │   │  10 minutes      │
└────────┬─────────┘   └────────┬─────────┘
         │                      │
         └──────────┬───────────┘
                    ▼
         ┌────────────────────┐
         │  FLIGHT_PLAN.md    │
         │  (this file)       │
         │                    │
         │  The only channel. │
         │  No terminal ctrl. │
         │  No direct cmds.   │
         └────────────────────┘
```

**Why no direct control:** Cowork cannot type into Terminal (platform safety boundary — Terminal is click-tier only). This is not a workaround — the Dual-Patrol architecture is inherently more resilient than direct orchestration. Each agent can fail independently without corrupting the other's state.

**There is no fallback channel.** If file-based communication fails (agent not reading, stuck, crashed), the response is ESCALATE_TO_HUMAN. The human director restarts the stuck agent, which then reads this file to restore context.

---

## 📝 OPTIONAL — OBSIDIAN VAULT FOR CROSS-SESSION MEMORY

Obsidian vault is recommended for cross-session memory but not required for the core Dual-Patrol loop. Both agents can operate effectively using FLIGHT_PLAN.md alone.

If you maintain an Obsidian vault:
- Cowork reads `_Orchestration/decisions-log.md` on session start to restore context
- Cowork writes key decisions to the vault during the session
- When a session ends, write a `session-handoffs.md` entry for the next session's context
- Vault is optional — FLIGHT_PLAN.md is the single source of truth

---

## 🗂️ REQUIRED FOLDER MOUNTS
> Cowork must have these folders mounted at session start. Without them, agents cannot communicate.

| Mount Path | Purpose | Required By |
|------------|---------|-------------|
| `[ProjectRoot]/` | Project source code | All agents |
| `[ProjectRoot]/FLIGHT_DECK/FLIGHT_PLAN.md` | Shared communication hub | All agents |
| `[ProjectRoot]/FLIGHT_DECK/AUTOPILOT.md` | Claude Code instructions | Claude Code, Second Builder |
| `[ProjectRoot]/PRD.md` | Product requirements | All agents |
| `[ProjectRoot]/AUDIT_SCREENSHOTS/` | Test evidence | Cowork Opus |
| `~/YourProject-SecondBrain/` | Obsidian vault (persistent memory) | Cowork Opus |

**Pre-flight checklist before starting any session:**
1. ✅ Project folder mounted in Cowork
2. ✅ Obsidian vault folder mounted in Cowork
3. ✅ `FLIGHT_DECK/FLIGHT_PLAN.md` exists in project
4. ✅ `CLAUDE.md` exists in project root (imports FLIGHT_DECK/AUTOPILOT.md)
5. ✅ `FLIGHT_DECK/PREFLIGHT_CHECK.md` exists (testing depth and persona settings)
6. ✅ `FLIGHT_DECK/TEST_FLIGHT_PROTOCOL.md` exists (screen-by-screen execution protocol)
7. ✅ Claude Code / Second Builder sessions can read/write the project folder
8. ✅ AUDIT_SCREENSHOTS/ folder exists

---

## 💰 MODEL-TIER ROUTING
> Use the cheapest model that can do the job. Reasoning for testing. Speed for monitoring.

| Agent | Default Model | Role | Cost |
|-------|---------------|------|------|
| Cowork Opus (Brain + Patrol + Test Pilot) | **Opus 4.6** | 10-min patrol, deep testing, all decisions | High |
| Test Pilot Personas (if using persona subagents) | **Sonnet 4.6** | Behavioral testing with specific constraints | Medium |
| Claude Code (Builder) | **Sonnet 4.6** | Standard building — good balance of speed and quality | Medium |
| Second Builder (Builder) | **Sonnet 4.6** | Standard building — plan-first discipline | Medium |

### Opus Does Both Patrol and Testing

In the simplified design, Cowork runs as **Opus only.** Opus handles two modes:

- **Patrol mode:** Lightweight file check every 10 minutes. Low token cost (~500-1000 tokens per check).
- **Test mode:** Full UX testing with knowledge tiers. Higher token cost (~20,000-50,000 tokens per test run).

This means one model handles both lightweight patrol and deep testing. No separate agents needed.

### Cost Summary

| Scenario | What's Running | Approximate Cost |
|----------|---------------|-----------------|
| Build only (CC builds, Opus patrols) | 2 sessions | ~2x single session |
| Build + test pilot (3 knowledge tiers) | 2 sessions + test runs | ~3-4x single session |
| Overnight operation (Ralph loop + Opus patrol + testing) | 2 sessions, continuous | ~5-7x single session |

**Budget guidance:** On Max subscription, budget ~4-6 hours of overnight operation. Opus patrol checks are cheap; the expensive part is the full test runs.

---

## 📊 STATUS BLOCK
> Both agents read this section FIRST on every patrol.

```
CURRENT_PHASE:  [build | test | fix | retest | complete | failed]
ASSIGNED_TO:    [claude_code | cowork | human]
STATUS:         [see STATUS CODES below]
LAST_UPDATED:   [YYYY-MM-DD HH:MM]
UPDATED_BY:     [claude_code | cowork | human]
ITERATION:      [1, 2, 3...]
```

---

## STATUS CODES

### Claude Code reads these — they trigger builder actions

| Code | Meaning | What Claude Code does |
|------|---------|----------------------|
| `BUILD_REQUESTED` | Initial build or rebuild needed | Run build command. If success, launch app, set `READY_FOR_TEST`. If fail, log error in FINDINGS, keep status. |
| `FIX_REQUESTED` | Cowork found bugs | Read FINDINGS (open items). Fix each bug. Rebuild. If success, set `READY_FOR_TEST`. If fix is unclear, write question in ACTIONS_TAKEN, set `ESCALATE_TO_HUMAN`. |
| `RETEST_PASSED` | All tested items passed | Do nothing. Wait for Cowork to advance to next persona or set `LOOP_COMPLETE`. |
| `LOOP_COMPLETE` | All personas passed | Do nothing. Celebration. |

### Cowork reads these — they trigger testing actions

| Code | Meaning | What Cowork does |
|------|---------|-----------------|
| `READY_FOR_TEST` | App is built and running | Open the app via computer use. Run ACTIVE_PERSONA test. Evaluate all screens/flows. Write results to FINDINGS. If bugs found, set `FIX_REQUESTED`. If all pass, set `RETEST_PASSED`. |
| `RETEST_REQUESTED` | Fixes applied, verify them | Open the app. Retest ONLY the items marked `fixed` in FINDINGS. If all verified, mark them `verified`, set `RETEST_PASSED`. If any still broken, update FINDINGS, set `FIX_REQUESTED`. |
| `AWAITING_TESTER` | App idle, waiting for pickup | Same as `READY_FOR_TEST` — begin testing. |

### Both agents read these — they stop normal work

| Code | Meaning | What both agents do |
|------|---------|-------------------|
| `ESCALATE_TO_HUMAN` | Timeout exceeded or agent stuck | Stop all work. Write current state to ACTIONS_TAKEN. Wait for human to update STATUS. |
| `LOOP_COMPLETE` | Ship it | Stop. All personas passed. Log final summary to ACTIONS_TAKEN. |

---

## NEXT ACTION

> **Plain language instructions. The agent who updates STATUS also writes the next action for the other agent.**
> This eliminates ambiguity — each agent knows exactly what to do on pickup.

```
NEXT_ACTION_FOR_CLAUDE_CODE:
  [e.g., "Fix the 3 bugs in FINDINGS items 1-3. The name parser regex
   (item 1) is highest priority — see root cause in the description.
   Rebuild with xcodebuild, launch the app, then set STATUS to
   READY_FOR_TEST."]

NEXT_ACTION_FOR_COWORK:
  [e.g., "App is rebuilt with fixes for items 1-3. Retest only those
   items: verify Sharick name displays without parenthetical, verify
   context menu has 5 items, verify undo toast appears. Update FINDINGS
   status for each. Set STATUS to RETEST_PASSED or FIX_REQUESTED."]
```

### Writing good NEXT_ACTION prompts

**Be specific.** Not "fix the bugs" — instead "fix the regex in NameParser.swift line 51 that fails on unclosed parentheses from OCR truncation."

**Reference FINDINGS by number.** "Fix items 2 and 5" not "fix the issues."

**State the completion signal.** Always end with "then set STATUS to X."

**Include verification steps.** For Cowork: "verify the name displays as 'Sharick, Louise M' without '(Lou'." For Claude Code: "build should complete with 0 errors. Warnings OK."

---

## FINDINGS

> **Cowork writes. Claude Code reads and fixes. Cowork verifies.**
> Each finding has a lifecycle: `open` → `fixed` → `verified` (or back to `open`).

```
[1] severity: major | context_menu_missing_items
    CH09 R4: Right-click context menu missing "Set Description" and
    "Select All in Group". Only shows Assign/Exclude/Transform.
    File: PhotoGridView.swift
    found_by: cowork | status: open

[2] severity: major | parenthetical_name_not_stripped
    CH05: OCR truncated "(Lou" without closing paren not stripped by
    regex in NameParser.swift:51. Regex requires closing ) but OCR
    doesn't always provide it.
    found_by: cowork | status: open

[3] severity: minor | description_placeholder
    ...
    found_by: cowork | status: open
```

### Severity levels

| Level | Meaning | Fix priority |
|-------|---------|-------------|
| `critical` | App crashes, data loss, blocks entire flow | Fix immediately, before any other work |
| `major` | Feature broken or spec violation, but app doesn't crash | Fix in current iteration |
| `minor` | Cosmetic, non-blocking, nice-to-have | Fix if time permits, or defer |

### Finding status lifecycle

```
open ──→ fixed (Claude Code applied fix and rebuilt)
              ──→ verified (Cowork retested and confirmed)
              ──→ open (Cowork retested and it's still broken)
```

---

## PROJECT INFO

```
PROJECT_NAME:       [e.g., PhotoSortVision]
PROJECT_PATH:       [e.g., /Users/huan/Desktop/PhotoSortVision]
BUILD_TOOL:         [e.g., Xcode]
BUILD_COMMAND:      [e.g., xcodebuild -scheme PhotoSortVision -configuration Debug build]
APP_LAUNCH:         [e.g., open /path/to/PhotoSortVision.app]
PRD_PATH:           [e.g., CLAUDE.md + CH01-CH17 chapter specs]
PATROL_INTERVAL:    10 minutes
TIMEOUT_WINDOW_1:   60 minutes
TIMEOUT_WINDOW_2:   30 minutes
```

### APP BUILD & LAUNCH

**For Xcode projects:** Cowork can build via GUI (Product → Clean Build Folder → Product → Build → Product → Run). Terminal is click-tier only but Xcode menu clicks work. Claude Code can also build via `xcodebuild` command line.

**APP_BUILD_METHOD:** [xcodebuild CLI | Cowork clicks Xcode GUI | npm run dev | etc.]
**APP_LAUNCH:** [open /path/to/App.app | Cowork clicks Run in Xcode | localhost:3000]

---

## PATROL PROTOCOL

### For Claude Code (Builder)

```
Every PATROL_INTERVAL (10 minutes):
  1. Read CURRENT STATUS BLOCK
  2. If STATUS is BUILD_REQUESTED or FIX_REQUESTED:
     → Read NEXT_ACTION_FOR_CLAUDE_CODE
     → Execute the instructions
     → Log result to ACTIONS_TAKEN
     → Update STATUS (typically to READY_FOR_TEST)
     → Write NEXT_ACTION_FOR_COWORK describing what was done
     → Update LAST_UPDATED and UPDATED_BY
  3. If STATUS is anything else:
     → Do nothing. Log patrol check if desired.
  4. Check TIMEOUT: if your current task has been running longer
     than TIMEOUT_WINDOW_1 without completion:
     → Pause. Retry once.
     → If retry exceeds TIMEOUT_WINDOW_2, set ESCALATE_TO_HUMAN.
```

### For Cowork (Test Pilot)

```
Every PATROL_INTERVAL (10 minutes):
  1. Read CURRENT STATUS BLOCK
  2. If STATUS is READY_FOR_TEST, RETEST_REQUESTED, or AWAITING_TESTER:
     → Read NEXT_ACTION_FOR_COWORK
     → Open the app via computer use
     → Test according to ACTIVE_PERSONA and instructions
     → Write results to FINDINGS
     → Log actions to ACTIONS_TAKEN
     → Update STATUS (FIX_REQUESTED or RETEST_PASSED)
     → Write NEXT_ACTION_FOR_CLAUDE_CODE with specific fix instructions
     → Update LAST_UPDATED and UPDATED_BY
  3. If STATUS is anything else:
     → Do nothing. Log patrol check if desired.
  4. Check TIMEOUT: if your current task has been running longer
     than TIMEOUT_WINDOW_1 without completion:
     → Pause. Retry once.
     → If retry exceeds TIMEOUT_WINDOW_2, set ESCALATE_TO_HUMAN.
```

### Patrol timing

```
PATROL_INTERVAL:    10 minutes (default, configurable)
TIMEOUT_WINDOW_1:   60 minutes — if task not complete, pause and retry once
TIMEOUT_WINDOW_2:   30 minutes — if retry fails, set ESCALATE_TO_HUMAN
```

Both agents should note their patrol in ACTIONS_TAKEN only when they take action (not on empty polls). This keeps the log clean.

---

## 🤖 AGENT ROLES & RULES

### Claude Code (Builder)

- Runs in Terminal
- Reads FLIGHT_PLAN.md STATUS BLOCK on every patrol (every 10 minutes)
- When STATUS is `BUILD_REQUESTED` or `FIX_REQUESTED`:
  - Reads NEXT_ACTION_FOR_CLAUDE_CODE
  - Executes the build/fix instructions
  - Logs results to ACTIONS_TAKEN
  - Updates STATUS to `READY_FOR_TEST` when complete
  - Writes NEXT_ACTION_FOR_COWORK
- When STATUS is anything else:
  - Does nothing. Waits.
- If stuck for >60 min (TIMEOUT_WINDOW_1):
  - Pauses, retries once
  - If still stuck after 30 more minutes, sets `ESCALATE_TO_HUMAN`

### Cowork (Test Pilot)

- Runs in Cowork desktop
- Reads FLIGHT_PLAN.md STATUS BLOCK on every patrol (every 10 minutes)
- When STATUS is `READY_FOR_TEST`, `RETEST_REQUESTED`, or `AWAITING_TESTER`:
  - Reads NEXT_ACTION_FOR_COWORK
  - Opens the app via computer use
  - Tests according to the ACTIVE_PERSONA
  - Writes bugs to FINDINGS with severity levels
  - Logs results to ACTIONS_TAKEN
  - Updates STATUS to either `FIX_REQUESTED` (bugs found) or `RETEST_PASSED` (all pass)
  - Writes NEXT_ACTION_FOR_CLAUDE_CODE with specific fix instructions
- When STATUS is anything else:
  - Does nothing. Waits.
- If stuck for >60 min (TIMEOUT_WINDOW_1):
  - Pauses, retries once
  - If still stuck after 30 more minutes, sets `ESCALATE_TO_HUMAN`

---

## TEST PERSONAS

> **Cowork works through personas one at a time. Each persona is a full test pass.**

```
ACTIVE_PERSONA:      [e.g., Cold User (Tier 1)]
PERSONA_SPEC:        [e.g., "Knows nothing about the app. No manual, no PRD."]
PERSONAS_COMPLETED:  [e.g., Cold User ✅, Guided User ✅]
PERSONAS_REMAINING:  [e.g., Insider (Tier 3)]
```

### Persona progression

When a persona's test pass is complete (all findings verified or no findings):
1. Move current persona to PERSONAS_COMPLETED
2. Set next persona as ACTIVE_PERSONA
3. If app changes were needed, set `STATUS: BUILD_REQUESTED` (Claude Code rebuilds before next persona)
4. If no changes needed, set `STATUS: READY_FOR_TEST` (Cowork continues with next persona)
5. When PERSONAS_REMAINING is empty, set `STATUS: LOOP_COMPLETE`

---

## ACTIONS TAKEN

> **Append-only log. Both agents write. Never delete entries.**

```
[2026-03-27 20:15] claude_code — Built successfully (21 warnings, 0 errors). App launched.
[2026-03-27 20:18] cowork — Tested Grandma Rose persona on Import and Review screens. 2 bugs found (items 1-2). Set FIX_REQUESTED.
[2026-03-27 20:25] claude_code — Fixed item 1 (PhotoGridView.swift +32 lines) and item 2 (NameParser.swift regex). Rebuilt. App relaunched. Set READY_FOR_TEST.
[2026-03-27 20:28] cowork — Retested items 1-2. Both verified. Set RETEST_PASSED.
```

---

## ESCALATION RULES

### Automatic escalation (either agent)

- Task exceeds TIMEOUT_WINDOW_1 + TIMEOUT_WINDOW_2 → `ESCALATE_TO_HUMAN`
- Build fails 3 consecutive times → `ESCALATE_TO_HUMAN`
- Same finding fails verification 3 times → `ESCALATE_TO_HUMAN`

### Claude Code escalation

- Unclear how to fix a finding → write question in ACTIONS_TAKEN, set `ESCALATE_TO_HUMAN`
- Architectural decision needed (not in PRD) → set `ESCALATE_TO_HUMAN`
- Fix requires changes to files outside project scope → set `ESCALATE_TO_HUMAN`

### Cowork escalation

- App won't launch or is unresponsive → set `ESCALATE_TO_HUMAN`
- Finding is ambiguous (might be intended behavior) → write question, set `ESCALATE_TO_HUMAN`
- Computer use can't access required UI element → log limitation, set `ESCALATE_TO_HUMAN`

### Human resolution

When human sees `ESCALATE_TO_HUMAN`:
1. Read ACTIONS_TAKEN for context
2. Make the decision or fix the blocker
3. Update NEXT_ACTION for the appropriate agent
4. Set STATUS back to the appropriate active state
5. Agents pick up on next patrol
Test Pilot Result: ✅ PASS / ❌ FAIL / ⏭️ SKIPPED (Low risk)
Comments:
  [specific feedback, direction, or architectural guidance]
Next Action:
  [what the agent should do next]
```

```
[YYYY-MM-DD HH:MM] — Opus → Second Builder
Re: T002 (Plan Review)
Decision: PLAN APPROVED — PROCEED TO EXECUTE / REVISE PLAN
Comments:
  [feedback on the plan]
Next Action:
  [execute as planned / revise step 3 because...]
```

```
[YYYY-MM-DD HH:MM] — Opus → Claude Code (Batch Review)
Tasks Reviewed: T005, T006, T007 (all 🟢 Low auto-proceeded)
Batch Verdict: ✅ All acceptable / 🔁 T006 needs revision
Comments:
  [any issues found across the batch]
Next Action:
  [continue auto-proceeding / stop and fix T006]
```

---

## 🔁 THE COMBINED ENGINE — RALPH WIGGUM LOOP + TEST PILOT LOOP

> [Ralph Wiggum](https://github.com/anthropics/claude-code/blob/main/plugins/ralph-wiggum/README.md) is a Claude Code plugin that creates persistent build iteration loops — it feeds the same prompt to Claude Code repeatedly, with each iteration seeing the modified files from prior attempts, until a completion promise is met.
>
> Ralph Wiggum Loop is the **build engine** — persistent iteration until code works.
> Test Pilot Loop is the **validation gate** — behavioral testing until UX works.
> Cowork Opus is the **brain** that controls both loops.
> Together: code doesn't just compile — it works for real humans.

### How They Fit Together

Ralph Wiggum Loop alone: iterates until tests pass. Ships code that works but may confuse users.
Test Pilot Loop alone: validates UX beautifully but has no build engine.
**Combined under Cowork Opus: iterates until code passes AND real users can use it.**

```
┌─────────────────────────────────────────────────────────────────┐
│                    🟣 COWORK OPUS (THE BRAIN)                   │
│           Controls both loops. Makes all decisions.             │
│           Writes prompts. Reviews results. Test-pilots.         │
└──────────────────────────┬──────────────────────────────────────┘
                           │
         ┌─────────────────▼─────────────────┐
         │     🔄 RALPH WIGGUM LOOP          │
         │     (Build Iteration Engine)       │
         │                                    │
         │  while not DONE:                   │
         │    │                               │
         │    ├─ 1. Feed PROMPT.md to Claude  │
         │    │     Code / Second Builder              │
         │    │                               │
         │    ├─ 2. Agent builds feature      │
         │    │                               │
         │    ├─ 3. Code tests pass?          │
         │    │     NO → loop (Ralph retries) │
         │    │     YES ↓                     │
         │    │                               │
         │    ├─ 4. 🧪 TEST PILOT GATE       │
         │    │  ┌──────────────────────┐     │
         │    │  │ Launch app            │     │
         │    │  │ Test as 4 personas    │     │
         │    │  │ Score 6 dimensions    │     │
         │    │  │ Run 3 knowledge tiers │     │
         │    │  │                       │     │
         │    │  │ UX PASS? ─NO─→ feed  │     │
         │    │  │   findings back into  │     │
         │    │  │   PROMPT.md → Ralph   │     │
         │    │  │   loops again         │     │
         │    │  │                       │     │
         │    │  │ YES ↓                 │     │
         │    │  │ ✅ DONE              │     │
         │    │  └──────────────────────┘     │
         │    │                               │
         │  done                              │
         └────────────────────────────────────┘
```

### The Key Innovation: Two Exit Gates, Not One

**Standard Ralph Wiggum:** exits when code tests pass (`<promise>COMPLETE</promise>`)
**Ralph + Test Pilot:** exits when code tests pass **AND** Test Pilot approves UX

```
COMPLETION CRITERIA (both must be true):
  ✅ Gate 1 (Ralph): Code compiles, tests pass, linter clean
  ✅ Gate 2 (Test Pilot): Persona testing passes, UX score ≥ threshold
  
  Only when BOTH gates pass → task is truly DONE
```

### How Cowork Opus Controls the Combined Loop

Opus doesn't just observe — it **writes the prompts** that drive Ralph and **reviews the results** that come out of Test Pilot.

**Before the loop starts (Opus as architect):**
1. Opus reads PRD → decomposes into tasks with risk levels
2. Opus writes `PROMPT.md` for each task with:
   - Clear requirements from PRD
   - Success criteria (code + UX)
   - Completion promise text
   - Max iteration limit (safety net)
   - Test Pilot gate criteria (which personas, which dimensions, threshold score)

**During the loop (Opus as supervisor):**
3. Ralph feeds prompt to Claude Code / Second Builder
4. Agent builds → code tests pass → triggers Test Pilot gate
5. Opus (or Opus patrol for ⚪/🟢) runs Test Pilot validation
6. If Test Pilot FAILS: Opus appends findings to `PROMPT.md` and Ralph loops again
7. If Test Pilot PASSES: Opus marks task ✅, moves to next

**After the loop (Opus as reviewer):**
8. Opus writes results to Obsidian vault
9. Opus updates UX trend dashboard
10. Opus moves to next task in queue

### Ralph Loop Configuration Per Risk Level

| Risk Level | Ralph Max Iterations | Test Pilot Gate | Who Reviews |
|------------|---------------------|-----------------|-------------|
| ⚪ Trivial | 3 | None — Opus patrol fast-approve | Opus (patrol) |
| 🟢 Low | 10 | Quick test (Expert persona only) | Opus batch review |
| 🟡 Medium | 20 | Standard test (3 personas, 6 dimensions) | Opus per-task |
| 🔴 High | 30 | Full test (all personas + domain + knowledge tiers) | Opus + human approval |
| 🏁 Milestone | 50 | Comprehensive (all personas, full journey, regression) | Opus + human director |

### What Gets Fed Back Into Ralph When Test Pilot Fails

When the Test Pilot finds UX problems, the findings are structured and appended to the prompt for the next Ralph iteration:

```
# PROMPT.md (updated by Opus after Test Pilot failure)

## Original Task
[original requirements]

## Test Pilot Findings (Iteration 3)
The following UX issues were found. Fix them in this iteration:

BUG-T004-001 [S3 Major]: "Import" button requires double-tap
  → Likely event handler conflict with initialization
  → Fix: ensure tap handler is attached after component is ready

BUG-T004-002 [S4 Minor]: Progress bar doesn't animate during import
  → CSS animation not triggering on state change

UX FEEDBACK from Everyday User persona:
  → "I didn't know which button to press first"
  → Add instructional text above the import button

UX FEEDBACK from Struggling User persona:
  → "The button is too small for my fingers"
  → Increase touch target to minimum 44x44pt

## Updated Success Criteria
- All original requirements still apply
- PLUS: all Test Pilot bugs above must be fixed
- PLUS: UX score must reach ≥ 20/30 (currently 16/30)

Output <promise>COMPLETE</promise> when done.
```

### Overnight Autonomous Mode (Ralph + Test Pilot + Cowork)

The combined system can run overnight with Opus in autopilot:

```bash
# Opus writes all PROMPT.md files for the night's work
# Then launches Ralph loops in sequence

# Terminal 1: Feature build + test pilot
/ralph-loop "$(cat PROMPT_T001.md)" \
  --max-iterations 20 \
  --completion-promise "COMPLETE"

# After T001 completes, Opus test-pilots the result
# If pass → next task. If fail → Ralph loops again with findings.

# Terminal 2: Parallel feature (different module)
/ralph-loop "$(cat PROMPT_T002.md)" \
  --max-iterations 20 \
  --completion-promise "COMPLETE"
```

Opus patrols every 10 minutes even during overnight runs. Kill switch still works. Confidence scores still trigger auto-stop.

---

## 🔄 AGENT WORKFLOW — THE FEEDBACK LOOP

### Claude Code Workflow (with Autopilot + Confidence)
```
0.  CHECK GLOBAL STOP — if 🛑 FROZEN → do nothing
1.  Opus assigns task → CC reads task + risk level + FEEDBACK QUEUE
2.  CC writes plan to AGENT OUTPUT LOG
3.  CC executes plan
4.  CC writes output + CONFIDENCE SCORE to AGENT OUTPUT LOG
5.  ┌─ IF ⚪ Trivial:
    │   CC sets status → ⏳ Awaiting Review
    │   Opus fast-approves on next patrol check (Opus NOT woken)
    │
    ├─ IF 🟢 Low risk AND confidence ≥ 80% AND next task is also 🟢 Low:
    │   CC sets status → 🟢 Auto-Proceeding
    │   CC starts next task immediately (NO WAIT)
    │   Opus patrol checks every 10 min, flags for batch review after 3+ tasks
    │
    ├─ IF 🟢 Low risk BUT confidence < 80%:
    │   CC STOPS → sets ⏳ Awaiting Review (confidence too low to auto-proceed)
    │
    └─ IF 🟡 Medium or 🔴 High risk:
        CC sets status → ⏳ Awaiting Review
        CC STOPS ← ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
                                                    ↑
        Cowork Opus reviews output                  │ CC waits here
        Opus test-pilots the app (🟡/🔴)            │ until feedback
        Opus writes to FEEDBACK QUEUE               │ arrives
        CC reads feedback                           │
        CC proceeds based on direction ← ━━━━━━━━━━┘
```

### Second Builder Workflow (Plan-First, Always Full Stop)
```
1.  Opus assigns task → initiates Second Builder in PLAN MODE ONLY
2.  Second Builder writes plan to AGENT OUTPUT LOG
3.  Second Builder sets status → 📋 Planning
4.  Second Builder STOPS ← ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
5.                                              ↑
6.  Cowork Opus reviews plan                    │ Second Builder waits here
7.  Opus writes to FEEDBACK QUEUE:              │ until plan is
8.    "PLAN APPROVED — PROCEED" or "REVISE"     │ approved
9.  Second Builder reads feedback ← ━━━━━━━━━━━━━━━━━━━━┘
10. If approved: Second Builder executes → output → ⏳ Awaiting Review → STOPS
11. Opus reviews execution output
12. Opus test-pilots the app (if 🟡/🔴)
13. Opus writes feedback → next cycle
```

### Cowork Opus Full Cycle (Review + Test Pilot)
```
1.  WAKE (Opus patrol trigger or human manual trigger)
2.  Read STATUS BLOCK for current state
3.  Read AGENT OUTPUT LOG for all pending ⏳ items
4.  Read Obsidian vault for persistent context
5.  Review each pending item:
    ┌─ For 🟡/🔴 tasks:
    │   Review code → Test-pilot the app → Write test pilot report
    │   If PASS → approve + feedback. If FAIL → revision needed + bugs.
    │
    └─ For 🟢 batch reviews:
        Scan auto-proceeded outputs for errors
        Spot-check 1-2 items by test-piloting
        Approve batch or flag specific items
6.  Write feedback/direction into FLIGHT_PLAN.md
    • Update FEEDBACK QUEUE with direction for each agent
    • Update TASK QUEUE if needed (add/modify/reassign)
7.  Update NEXT_ACTION_FOR_CLAUDE_CODE with specific fix instructions:
    • "Read FINDINGS items 1-3 in FLIGHT_PLAN.md. Fix the issues,
       rebuild, relaunch, and set STATUS to READY_FOR_TEST."
    • Set STATUS to FIX_REQUESTED
    • Claude Code picks this up on its next 10-minute patrol
8.  Write decisions to Obsidian vault if configured (decisions-log.md)
9.  Return to patrol mode (10-minute checks)
```

---

## 📝 AGENT OUTPUT LOG
> Append only. Never overwrite. Always include timestamp + agent name.
> After writing output, agent sets status based on risk level.

```
[YYYY-MM-DD HH:MM] — Claude Code
Task ID: T001
Risk Level: 🟢 Low
Plan:
  • Step 1:
  • Step 2:
Output:
  [result / error / code changes made]
Files Modified:
  [list of files touched]
Confidence: [0-100%]  ← How sure is CC that this is correct?
Status: 🟢 Auto-Proceeding to T002
Self-Assessment:
  [if confidence < 80%, explain why and consider stopping]
```

```
[YYYY-MM-DD HH:MM] — Claude Code
Task ID: T004
Risk Level: 🟡 Medium
Plan:
  • Step 1:
  • Step 2:
Output:
  [result / error / code changes made]
Files Modified:
  [list of files touched]
Confidence: [0-100%]  ← How sure is CC that this is correct?
Status: ⏳ Awaiting Review + Test Pilot
Self-Assessment:
  [if confidence < 80%, flag specific concerns for Opus]
```

```
[YYYY-MM-DD HH:MM] — Second Builder
Task ID: T002
Mode: PLAN ONLY
Plan:
  • Step 1:
  • Step 2:
  • Step 3:
Estimated Complexity: [Low / Medium / High]
Questions for Opus:
  [any architectural questions before execution]
Status: 📋 Planning — Awaiting Plan Approval
```

```
[YYYY-MM-DD HH:MM] — Second Builder
Task ID: T002
Mode: EXECUTION (Plan approved [timestamp])
Output:
  [result / error / code changes made]
Files Modified:
  [list of files touched]
Confidence: [0-100%]  ← How sure is Second Builder that this matches the approved plan?
Status: ⏳ Awaiting Review + Test Pilot
Self-Assessment:
  [deviations from plan, if confidence < 80% explain why]
```

---

## 🔍 AUDIT LOG
> Cowork Opus (test pilot), and Claude Code (code auditor) append findings here

```
[YYYY-MM-DD HH:MM] — Cowork Opus Patrol Check
Feature Tested:
PRD Reference:
Expected Outcome:
Actual Outcome:
UX Feedback: Smooth ✅ · Clunky ⚠️ · Broken ❌
Screenshot: [path/to/screenshot.png]
Action Required: None · Fix needed · Escalate to Opus
```

```
[YYYY-MM-DD HH:MM] — Cowork Opus Test Pilot
(See TEST PILOT REPORT template above)
```

```
[YYYY-MM-DD HH:MM] — Claude Code Audit
File/Module:
Issues Found:
  • Issue 1
  • Issue 2
Recommendation:
Action Required: None · Fix needed · Escalate to Opus
```

---

## ⏱️ OPUS 10-MINUTE PATROL LOG
> Opus patrol checks ALL agents every 10 minutes. No exceptions.

```
[YYYY-MM-DD HH:MM] — Opus Patrol #001

GLOBAL STOP: ACTIVE / 🛑 FROZEN

AGENT STATUS REPORT:
  • Claude Code:
    - Current status: [🔄 building / 🟢 auto-proceeding / ⏳ awaiting / idle]
    - Current task: [T00X]
    - Tasks completed since last patrol: [count]
    - Auto-proceeded tasks pending batch review: [count]
    - Confidence scores since last patrol: [T005: 95%, T006: 88%, T007: 72% ⚠️]
    - Errors detected: [none / describe]
    - Health: OK / ⚠️ CONCERN / ❌ STUCK

  • Second Builder:
    - Current status: [📋 planning / 🔄 executing / ⏳ awaiting / idle]
    - Current task: [T00X]
    - Time in current state: [X min]
    - Last confidence score: [X%]
    - Health: OK / ⚠️ CONCERN / ❌ STUCK

  • Cowork Opus Test Status:
    - Current status: [testing / idle / reporting]
    - Last test completed: [timestamp]
    - Open findings: [count]
    - Health: OK / ⚠️ CONCERN / ❌ STUCK

  • Cowork Opus:
    - Current status: [awake / reviewing / test-driving / sleeping]
    - Last feedback written: [timestamp]
    - Pending reviews: [count of ⏳ items]

FAST-APPROVALS THIS PATROL:
  - [T010 (⚪ Trivial) — comment cleanup — ✅ Opus patrol-approved]
  - [T011 (⚪ Trivial) — import reorder — ✅ Opus patrol-approved]
  - [or: None this patrol]

CONFIDENCE ALERTS:
  - [T007: CC reported 72% confidence — ESCALATING TO OPUS]
  - [or: All confidence scores ≥ 80% — no alerts]

ROUTING ACTIONS TAKEN:
  - [e.g., "Initiated Second Builder for T002 in plan-only mode"]
  - [e.g., "Flagged CC for batch review — 4 auto-proceeded tasks"]
  - [e.g., "Set ESCALATE OPUS: YES — T003 blocked for 12 min"]
  - [e.g., "Fast-approved T010, T011 (⚪ Trivial)"]

ESCALATION: YES / NO
  Reason: [if YES, why]

NEXT PATROL: [YYYY-MM-DD HH:MM]
```

---

## 🧠 OPUS DECISION LOG
> Opus writes here after waking. Also mirrors key decisions to Obsidian vault.

```
[YYYY-MM-DD HH:MM] — Opus Decision #001
Wake Trigger: [Opus patrol trigger / Human manual]
Context Loaded: [which Obsidian notes read]

Items Reviewed:
  • T001 (Claude Code, 🟡 Medium) — [reviewed code + test-piloted app → approved / revision]
  • T002 (Second Builder plan) — [plan approved / plan needs revision]
  • T005-T007 (Claude Code, 🟢 Low batch) — [batch approved / T006 flagged]

Test Pilots Performed:
  • T001: [PASS / FAIL — summary]
  • T004: [PASS / FAIL — summary]

Decision Summary:
  [key decisions made and reasoning]

Feedback Written:
  • Claude Code: [summary of feedback given]
  • Second Builder: [summary of feedback given]

Task Queue Changes:
  • [added T004: ... assigned to Claude Code, risk: 🟡 Medium]
  • [reassigned T003 from Second Builder to Claude Code — reason: ...]

Obsidian Updated:
  • decisions-log.md — [what was written]
  • [project]/current-state.md — [what was updated]

Returning to patrol mode: YES
Next expected wake: [when and why]
```

---

## 🚦 ESCALATION RULES

| Condition                                | Action                               |
|------------------------------------------|--------------------------------------|
| `GLOBAL STOP: 🛑 FROZEN`                | ALL agents stop immediately          |
| Any agent `⏳ Awaiting Review` (non-⚪)  | Opus enters full review mode      |
| Any agent `⏳ Awaiting Review` (⚪ only) | Opus fast-approves directly         |
| Agent confidence < 80%                   | Opus switches to full test mode immediately  |
| CC confidence 80-90% for 3+ tasks        | Opus flags for batch review    |
| Agent blocked > 10 min                   | Opus switches to full test mode              |
| 3+ audit failures on same feature        | Opus switches to full test mode              |
| Architecture or design decision needed   | Any agent sets `ESCALATE OPUS: YES`  |
| Claude Code and Second Builder outputs conflict   | Opus switches to full test mode              |
| All tasks complete                       | Opus begins final review + test pilot |
| Second Builder plan ready for review              | Opus enters full review mode      |
| CC auto-proceeded 3+ tasks               | Opus flags for batch review    |
| Human director triggers Cowork manually            | Cowork Opus wakes immediately        |
| Agent not responding after 2 patrol intervals | Set STATUS to ESCALATE_TO_HUMAN     |
| Agent stuck in loop / wrong direction    | Set STATUS to ESCALATE_TO_HUMAN     |
| Test pilot reveals critical bug          | Opus marks 🔴 and assigns immediate fix |
| System going in wrong direction          | Any agent or the human director sets `GLOBAL STOP: 🛑 FROZEN` |

---

## 🔒 WRITE PROTOCOL
> Prevent race conditions between agents

### File-Based Communication (Primary)
1. Before writing, agent appends `[AGENT NAME — WRITING]` as a single line
2. Complete the full entry before the next agent writes
3. Never delete or overwrite another agent's entry — append only
4. Only Opus (in patrol mode) updates the STATUS BLOCK (except `ESCALATE OPUS` which any agent can set to YES)
5. Only Cowork Opus updates the Task Queue
6. Only Cowork Opus writes to the FEEDBACK QUEUE
7. Agents must read FEEDBACK QUEUE before starting any new work

### No Direct Terminal Control
> Cowork cannot type into Terminal (click-tier safety boundary). All communication between agents happens exclusively through this file.

8. If an agent has not responded to STATUS changes within 2 patrol intervals (~20 minutes):
   - The other agent writes to ACTIONS_TAKEN: "[agent] appears unresponsive. Last update: [timestamp]."
   - Set STATUS to ESCALATE_TO_HUMAN
   - Human director restarts the stuck agent, which reads this file to resume
9. There is no fallback channel. FLIGHT_PLAN.md is the only communication mechanism.
10. If context is too complex for the NEXT_ACTION field (e.g., long error logs), write it to a separate file in the project and reference the path in NEXT_ACTION.

---

## CRASH RECOVERY

### Claude Code crashes or disconnects

Cowork will see stale LAST_UPDATED (no update within expected timeframe).
After 2 missed patrol intervals with no status change:
1. Cowork writes to ACTIONS_TAKEN: "Claude Code appears unresponsive. Last update: [timestamp]."
2. Cowork sets `ESCALATE_TO_HUMAN`
3. Human restarts Claude Code. Claude Code reads STATUS BLOCK, picks up where it left off.

### Cowork crashes or disconnects

Claude Code will see stale LAST_UPDATED after setting READY_FOR_TEST.
After 2 missed patrol intervals with no pickup:
1. Claude Code writes to ACTIONS_TAKEN: "Cowork has not picked up READY_FOR_TEST. Last update: [timestamp]."
2. Claude Code sets `ESCALATE_TO_HUMAN`
3. Human restarts Cowork. Cowork reads STATUS BLOCK, picks up where it left off.

### Both crash

Human notices both agents idle. Reads FLIGHT_PLAN.md for last known state. Restarts both. They resume from STATUS BLOCK.

---

## SETUP INSTRUCTIONS

### First time — human does this once

1. Copy this template into your project at `FLIGHT_DECK/FLIGHT_PLAN.md`
2. Fill in PROJECT INFO (name, path, build command, etc.)
3. Fill in TEST PERSONAS (which personas/tiers to run)
4. Set initial status:
   ```
   CURRENT_PHASE:  build
   ASSIGNED_TO:    claude_code
   STATUS:         BUILD_REQUESTED
   LAST_UPDATED:   [now]
   UPDATED_BY:     human
   ITERATION:      1
   ```
5. Write initial NEXT_ACTION:
   ```
   NEXT_ACTION_FOR_CLAUDE_CODE:
     "Build the app using [build command]. Launch it. Set STATUS
      to READY_FOR_TEST when the app is running."

   NEXT_ACTION_FOR_COWORK:
     "Wait for Claude Code to build. On next patrol, if STATUS is
      READY_FOR_TEST, begin testing with [first persona]."
   ```
6. Start Claude Code in Terminal with the project folder. Tell it:
   *"You are the Builder agent in the Test Pilot Loop. Read FLIGHT_DECK/FLIGHT_PLAN.md and follow the PATROL PROTOCOL for Claude Code. Check the file every 10 minutes."*
7. Start Cowork with the project folder mounted. Tell it:
   *"You are the Test Pilot in the Test Pilot Loop. Read FLIGHT_DECK/FLIGHT_PLAN.md and follow the PATROL PROTOCOL for Cowork. Check the file every 10 minutes."*
8. Both agents now run independently. Monitor ACTIONS_TAKEN for progress.

---

## CRITICAL RULES

1. **STATUS BLOCK is the single source of truth.** Both agents read it before every action.
2. **NEXT_ACTION is mandatory.** Every STATUS update must include plain-language instructions for the other agent.
3. **FINDINGS is append-only for new items.** Status field (open/fixed/verified) is updated in place.
4. **ACTIONS_TAKEN is append-only always.** Never delete log entries.
5. **One agent acts at a time.** STATUS determines who. The other waits.
6. **Never inflate progress.** If a fix didn't work, set it back to `open`. If a build failed, say so.
7. **Escalate early.** Stuck for more than one patrol cycle? Set `ESCALATE_TO_HUMAN`. Don't spin.
8. **This file is the only communication channel.** No Terminal commands between agents. No direct interaction. If it's not in this file, it didn't happen.

---

*Flight Plan — Dual-Patrol Model v0.2.0*
*Test Pilot Loop by Huan Su · 2026*
