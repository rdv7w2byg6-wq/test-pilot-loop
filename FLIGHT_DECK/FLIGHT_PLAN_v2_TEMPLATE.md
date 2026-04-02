# Flight Plan — Multi-Stage Coordination Document v2.0.0

> **Multiple agents. Shared files. Three stages. One heartbeat protocol.**
> Claude Code is the brain. Cowork Opus is the test pilot. Neither more, neither less.
> Communication happens through STATUS, NEXT_ACTION, FINDINGS, and the heartbeat cycle.

---

## 🏗️ ARCHITECTURE OVERVIEW

```
┌───────────────────────────────────────────────────────────┐
│                  HUMAN DIRECTOR (You)                      │
│   Starts agents. Sets budgets. Final authority.            │
│   Intervenes on ESCALATE_TO_HUMAN.                         │
│   Can rewind via Flight Recorder at any time.              │
└─────────────────────────┬─────────────────────────────────┘
                          │ launches agents
        ┌─────────────────┼──────────────────┐
        ▼                 ▼                  ▼
┌───────────────┐ ┌───────────────┐ ┌────────────────────┐
│  CLAUDE CODE  │ │  CODEX /      │ │  COWORK OPUS       │
│  (Brain +     │ │  GEMINI CLI   │ │  (Test Pilot ONLY) │
│   Lead        │ │  (Co-planners │ │                    │
│   Builder)    │ │   + optional  │ │  Tests app via     │
│               │ │   co-builders)│ │  computer use.     │
│  Runs in      │ │               │ │  Runs in Cowork.   │
│  Terminal.    │ │  Runs in      │ │  10-min heartbeat. │
│  fswatch      │ │  Terminal.    │ │                    │
│  heartbeat.   │ │  fswatch      │ │  Does NOT build.   │
│               │ │  heartbeat.   │ │  Does NOT decide.  │
│  Makes ALL    │ │               │ │  Does NOT patrol   │
│  decisions.   │ │  Debates plan │ │  for build tasks.  │
│  Owns STATUS  │ │  in Stage 1.  │ │                    │
│  BLOCK.       │ │  May co-build │ │  ONLY responds to  │
│               │ │  in Stage 2.  │ │  READY_FOR_TEST.   │
└───────┬───────┘ └───────┬───────┘ └─────────┬──────────┘
        │                 │                    │
        └────────┬────────┘                    │
                 ▼                             │
    ┌────────────────────────┐                 │
    │ FLIGHT_PLANNING/       │                 │
    │ DISCUSSION/            │                 │
    │ (Stage 1 only)         │                 │
    │ fswatch-triggered      │                 │
    └────────────────────────┘                 │
                                               │
        ┌──────────────────────────────────────┘
        │
        ▼
    ┌────────────────────────┐
    │ FLIGHT_PLAN.md         │
    │ (this file)            │
    │                        │
    │ Stage 2-3 channel.     │
    │ Build + Test signals.  │
    │ The primary channel.   │
    └────────────────────────┘
```

### Why Claude Code Is the Brain

Cowork cannot use `fswatch`, cannot run background processes, and cannot type into Terminal (platform safety boundary — Terminal is click-tier only). In v1, Cowork was both brain and tester, which overloaded its role and created reliability issues.

In v2, Claude Code handles all decision-making, coordination, and status transitions. Cowork does one thing only: test the running app through computer use and report findings.

**This is not a workaround — it matches each tool to its strength.** Claude Code is stable, CLI-based, and handles complex coordination well. Cowork's computer use capability is valuable for the one thing only it can do: interacting with a running application visually.

### Two Communication Channels

| Channel | Purpose | Who Uses It | Stage |
|---------|---------|-------------|-------|
| `FLIGHT_PLANNING/DISCUSSION/` | Multi-LLM deliberation on build files | Claude Code + Codex + Gemini CLI | Stage 1 only |
| `FLIGHT_PLAN.md` (this file) | Build-test coordination, status, findings | Claude Code + Cowork Opus | Stages 2 and 3 |

**There is no fallback channel.** If file-based communication fails (agent not reading, stuck, crashed), the response is ESCALATE_TO_HUMAN. The human director restarts the stuck agent, which reads this file to restore context.

---

## 🔄 THREE STAGES

### Stage 1: Flight Planning (Multi-LLM Deliberation)

**Optional — for new projects or major feature redesigns.**

Multiple AI coding agents (potentially from different providers) debate and refine the PRD, CLAUDE.md, state machine, and architecture files before any code is written.

See `FLIGHT_PLANNING/README.md` for the full deliberation protocol.

**This file (FLIGHT_PLAN.md) is NOT used during Stage 1.** Deliberation happens in `FLIGHT_PLANNING/DISCUSSION/`. When Stage 1 completes (all agents agree), the refined files in `FLIGHT_PLANNING/OUTPUT/` become the build spec for Stage 2.

### Stage 2: Flight (Building)

Claude Code builds the project using the refined plan. Optional co-builders (Codex, etc.) may assist. This is the active build phase — code is written, compiled, and launched.

**This file is the primary channel during Stage 2.** Claude Code writes status updates and signals readiness for testing.

### Stage 3: Test Flight

Cowork Opus tests the running application through computer use. Tests as different personas with different knowledge levels. Reports findings. Claude Code reads findings, fixes, and signals retest.

**This file is the primary channel during Stage 3.** Cowork writes test results and findings. Claude Code reads them and acts.

Stages 2 and 3 loop until all tests pass or the human director intervenes.

---

## 📝 OPTIONAL — OBSIDIAN VAULT FOR CROSS-SESSION MEMORY

Obsidian vault is recommended for cross-session memory but not required for the core loop. All agents can operate effectively using FLIGHT_PLAN.md and TEST_STATE.md alone.

If you maintain an Obsidian vault:

- Claude Code reads `_Orchestration/decisions-log.md` on session start to restore context
- Claude Code writes key decisions to the vault during the session
- When a session ends, write a `session-handoffs.md` entry for the next session's context
- Vault is optional — FLIGHT_PLAN.md is the single source of truth

---

## 🗂️ REQUIRED FOLDER MOUNTS

> Cowork must have these folders mounted at session start. Without them, agents cannot communicate.

| Mount Path | Purpose | Required By |
|------------|---------|-------------|
| `[ProjectRoot]/` | Project source code | All agents |
| `[ProjectRoot]/FLIGHT_DECK/FLIGHT_PLAN.md` | Build-test communication hub | Claude Code, Cowork Opus |
| `[ProjectRoot]/FLIGHT_DECK/AUTOPILOT.md` | Claude Code instructions | Claude Code, Second Builder |
| `[ProjectRoot]/FLIGHT_DECK/HEARTBEAT.md` | Unified heartbeat protocol | All agents |
| `[ProjectRoot]/FLIGHT_DECK/TEST_STATE.md` | Persistent test memory | Cowork Opus |
| `[ProjectRoot]/FLIGHT_DECK/BUDGET.md` | Spending caps and actuals | All agents |
| `[ProjectRoot]/FLIGHT_DECK/GOAL_ANCESTRY.md` | Mission → task tracing | All agents |
| `[ProjectRoot]/PRD.md` | Product requirements | All agents |
| `[ProjectRoot]/AUDIT_SCREENSHOTS/` | Test evidence | Cowork Opus |
| `~/YourProject-SecondBrain/` | Obsidian vault (persistent memory) | Optional |

**Pre-flight checklist before starting any session:**

1. ✅ Project folder mounted in Cowork
2. ✅ `FLIGHT_DECK/FLIGHT_PLAN.md` exists in project
3. ✅ `CLAUDE.md` exists in project root (imports FLIGHT_DECK/AUTOPILOT.md)
4. ✅ `FLIGHT_DECK/PREFLIGHT_CHECK.md` exists (testing depth and persona settings)
5. ✅ `FLIGHT_DECK/TEST_FLIGHT_PROTOCOL.md` exists (screen-by-screen execution protocol)
6. ✅ `FLIGHT_DECK/HEARTBEAT.md` exists (unified heartbeat protocol)
7. ✅ `FLIGHT_DECK/BUDGET.md` exists (spending caps configured)
8. ✅ `FLIGHT_DECK/GOAL_ANCESTRY.md` exists (mission and objectives defined)
9. ✅ Claude Code / Second Builder sessions can read/write the project folder
10. ✅ AUDIT_SCREENSHOTS/ folder exists
11. ✅ FLIGHT_RECORDER/ folder exists

---

## 💓 HEARTBEAT CONFIGURATION

> All agents follow the unified heartbeat protocol defined in `HEARTBEAT.md`.
> Same six steps: WAKE → READ → DECIDE → WRITE → BUDGET → SLEEP.

| Agent | Trigger | Watches | Role |
|-------|---------|---------|------|
| Claude Code (Brain + Lead Builder) | fswatch (instant) | `FLIGHT_PLAN.md` | Owns STATUS BLOCK, makes all decisions, builds, fixes |
| Codex / Gemini (Co-planner, optional co-builder) | fswatch (instant) | `DISCUSSION/` (Stage 1), `FLIGHT_PLAN.md` (Stage 2) | Debates plan, may co-build on tagged tasks |
| Cowork Opus (Test Pilot) | 10-min timer | `FLIGHT_PLAN.md` | Tests app via computer use, writes findings |

See `HEARTBEAT.md` for full setup commands, confirmation protocol, and escalation triggers.

**Neither agent is assumed running until it confirms in writing** in the AGENT OUTPUT LOG section below.

---

## 💰 MODEL-TIER ROUTING

> Use the cheapest model that can do the job. Reasoning for testing. Speed for building.

| Agent | Default Model | Role | Cost |
|-------|---------------|------|------|
| Claude Code (Brain + Lead Builder) | **Sonnet 4.6** | Building, decision-making, coordination | Medium |
| Second Builder (Codex / other) | **Sonnet-tier** | Co-building on tagged tasks | Medium |
| Cowork Opus (Test Pilot) | **Opus 4.6** | Deep UX testing with personas and knowledge tiers | High |
| Deliberation agents (Stage 1) | **Varies by provider** | Debating PRD, architecture, state machine | Varies |

### Why Opus for Testing

The Cold user persona testing requires genuine cognitive depth — Opus needs to authentically *not know* things and reason through confusion like a real confused human. A cheaper model would be too efficient at figuring things out, which defeats the purpose of measuring intuitiveness through the Tier Gap Ratio.

### Cost Summary

| Scenario | What's Running | Approximate Cost |
|----------|---------------|-----------------|
| Build only (CC builds, Opus on standby) | 1-2 sessions | ~1-2x single session |
| Build + test (3 knowledge tiers) | 2 sessions + test runs | ~3-4x single session |
| Full Flight Deck (planning + build + test) | 3+ sessions | ~5-8x single session |
| Overnight operation (Ralph loop + Opus testing) | 2 sessions, continuous | ~5-7x single session |

**Budget guidance:** Configure per-agent caps in `BUDGET.md`. On Max subscription, budget ~4-6 hours of overnight operation. See `BUDGET.md` for circuit breaker thresholds.

### Builder Agent Compatibility

The heartbeat model works with any builder agent that can read and write files:

| Builder Agent | Heartbeat Trigger | Notes |
|---------------|-------------------|-------|
| **Claude Code** | fswatch (instant) | Default lead builder. Owns STATUS BLOCK. |
| **Codex CLI** | fswatch (instant) | Drop-in co-builder. Acts on tasks tagged [CODEX]. |
| **Codex Cloud** | External wrapper | Needs CI script or GitHub Action to poll FLIGHT_PLAN.md and spawn tasks. |
| **OpenClaw** | Native event system | Becomes another builder agent via its own trigger system. |
| **Cursor / Windsurf** | Plugin-based | IDE agents may need a plugin to watch FLIGHT_PLAN.md. |

---

## 📊 STATUS BLOCK

> **All agents read this section FIRST on every heartbeat.**
> **Only Claude Code (the brain) may write to the STATUS BLOCK.** Cowork writes findings and feedback but does NOT change STATUS directly — it writes recommendations, and Claude Code updates STATUS on its next heartbeat.

```
GLOBAL_STOP:      [ACTIVE | 🛑 FROZEN]
PILOT_MODE:       [👨‍✈️ SUPERVISED | ✈️ AUTOPILOT | 🛬 AUTOPILOT-SAFE]
ACTIVE_STAGE:     [flight_planning | flight | test_flight | complete]
CURRENT_PHASE:    [build | test | fix | retest | complete | failed]
ASSIGNED_TO:      [claude_code | codex | cowork | human]
STATUS:           [see STATUS CODES below]
LAST_UPDATED:     [YYYY-MM-DD HH:MM]
UPDATED_BY:       [claude_code | cowork | human]
ITERATION:        [1, 2, 3...]
AUTOPILOT_UNTIL:  [YYYY-MM-DD HH:MM | Indefinite | N/A]
BUDGET_STATUS:    [NORMAL | ⚠️ WARNING (80%) | 🛑 CAPPED (100%)]
GOAL_CONTEXT:     [e.g., "Mission > Photo Import > Upload Flow"]
```

### 🛑 KILL SWITCH — GLOBAL STOP

**Any agent or the human director can set `GLOBAL_STOP: 🛑 FROZEN` at any time.**

When FROZEN:

- ALL agents immediately stop current work on their next heartbeat
- No new tasks may be started
- No status transitions except `ESCALATE_TO_HUMAN`
- Agents write their current state to AGENT OUTPUT LOG with note "🛑 Frozen"
- Heartbeat continues but takes NO actions — only logs status
- Only Claude Code (the brain) or the human director can set `GLOBAL_STOP: ACTIVE` to resume

When to freeze:

- System going in wrong architectural direction
- Agents caught in infinite loop or cascading errors
- Need to rethink approach before more work is wasted
- Any agent detects output that conflicts with PRD
- Human director stepping away and wants agents stopped
- Budget hits 100% on any agent (auto-freeze)
- Regression detected — Flight Recorder rollback may be needed

### ✈️ PILOT MODE — Human Supervision Level

The human director sets this before stepping away. Controls how much autonomy the agents have.

**👨‍✈️ SUPERVISED (Default):** Human director is available. Normal operations. Agents escalate per the escalation rules. Human approves critical bugs, design decisions, priority conflicts.

**✈️ AUTOPILOT:** Human director is away. Claude Code (the brain) has full authority.

- Claude Code makes ALL decisions that would normally escalate to human
- Handles critical bugs on its own — chooses the safest fix approach
- Makes design/UX decisions based on PRD and existing patterns
- **Guardrails still active:** Kill switch can still fire. Budget caps enforced. Confidence < 60% on any decision → Claude Code holds the task for human's return. No irreversible actions.
- All decisions logged in AGENT OUTPUT LOG with `[AUTOPILOT DECISION]` tag
- When `AUTOPILOT_UNTIL` time is reached, mode reverts to SUPERVISED automatically
- Flight Recorder checkpoint created before every AUTOPILOT DECISION

**🛬 AUTOPILOT-SAFE (Conservative):** Human director is away but wants minimal risk.

- Agents can only proceed with minor/trivial findings
- Critical or major findings → Claude Code sets `GLOBAL_STOP: 🛑 FROZEN` and waits
- No architectural decisions, no design changes
- Essentially: "keep doing easy work, stop if anything hard comes up"

---

## 🚦 GOVERNANCE GATES

> Stage transitions require explicit conditions. They can be automatic (overnight) or human-approved (supervised).

| Gate | From → To | Automatic Condition | Human Override |
|------|-----------|---------------------|----------------|
| **Planning Complete** | Flight Planning → Flight | All agents AGREE in AGREEMENT.md | Director approves plan early |
| **Build Ready** | Flight → Test Flight | Code compiles/runs + Claude Code signals READY_FOR_TEST | Director triggers test at any point |
| **Test Passed** | Test Flight → Complete | All required persona/tier combos pass | Director accepts with known issues |
| **Test Failed** | Test Flight → Flight | Any persona/tier combo fails | Director can override to Complete |
| **Rethink** | Test Flight → Flight Planning | Fundamental design issue found in testing | Director decides to revisit plan |
| **Rollback** | Any stage → Previous checkpoint | Regression detected or Claude Code requests | Director rewinds via Flight Recorder |
| **Budget Cap** | Any stage → Frozen | Any agent at 100% budget | Director increases budget to resume |

### Gate Logging

Every gate transition is logged to `FLIGHT_LOG.md` (append-only):

```
[timestamp] GATE_PASS | Build Ready → Test Flight (automatic) | iteration: 3
[timestamp] GATE_PASS | Test Failed → Flight (automatic) | Cold user failed upload
[timestamp] GATE_PASS | Rollback → checkpoint_005 (human) | sort fix caused regression
```

---

## STATUS CODES

### Claude Code reads these — they trigger builder actions

| Code | Meaning | What Claude Code does |
|------|---------|----------------------|
| `BUILD_REQUESTED` | Initial build or rebuild needed | Run build command. If success, launch app, set `READY_FOR_TEST`. If fail, log error in FINDINGS, keep status. Create Flight Recorder checkpoint. |
| `FIX_REQUESTED` | Test pilot found bugs | Read FINDINGS (open items). Fix each bug. Rebuild. If success, set `READY_FOR_TEST`. If fix is unclear, write question in AGENT OUTPUT LOG, set `ESCALATE_TO_HUMAN`. |
| `RETEST_PASSED` | All tested items passed | Check if all persona/tier combos complete. If yes, set `LOOP_COMPLETE`. If more combos remaining, update ACTIVE_PERSONA/TIER. Set `READY_FOR_TEST`. |
| `LOOP_COMPLETE` | All personas passed | Create final Flight Recorder checkpoint. Write completion summary. |

### Cowork reads these — they trigger testing actions

| Code | Meaning | What Cowork does |
|------|---------|-----------------|
| `READY_FOR_TEST` | App is built and running | Read TEST_STATE.md for context from previous cycles. Open the app via computer use. Run ACTIVE_PERSONA test per TEST_FLIGHT_PROTOCOL.md. Write results to FINDINGS. Write recommended status to AGENT OUTPUT LOG (e.g., "Recommend FIX_REQUESTED — 3 bugs found"). |
| `RETEST_REQUESTED` | Fixes applied, verify them | Read TEST_STATE.md. Retest ONLY the items marked `fixed` in FINDINGS. Update FINDINGS status. Write recommended status to AGENT OUTPUT LOG. Update TEST_STATE.md with progress. |
| `AWAITING_TESTER` | App idle, waiting for pickup | Same as `READY_FOR_TEST` — begin testing. |

**Important change from v1:** Cowork writes *recommendations*, not status changes. Claude Code reads the recommendation on its next heartbeat and updates the STATUS BLOCK. This ensures one agent (Claude Code) owns all state transitions, preventing race conditions.

### Both agents read these — they stop normal work

| Code | Meaning | What both agents do |
|------|---------|---------------------|
| `ESCALATE_TO_HUMAN` | Stuck, timeout, or low confidence | Stop all work. Write current state to AGENT OUTPUT LOG. Wait for human to update STATUS. |
| `LOOP_COMPLETE` | Ship it | Stop. All personas passed. Log final summary. |

---

## NEXT ACTION

> **Plain language instructions. Claude Code writes both — the action for itself and the action for Cowork.**
> This eliminates ambiguity — each agent knows exactly what to do on pickup.

```
NEXT_ACTION_FOR_CLAUDE_CODE:
  [e.g., "Fix the 3 bugs in FINDINGS items 1-3. The name parser regex
   (item 1) is highest priority — see root cause in the description.
   Rebuild with xcodebuild, launch the app, then set STATUS to
   READY_FOR_TEST. Goal context: Mission > Photo Import > Name Display."]

NEXT_ACTION_FOR_COWORK:
  [e.g., "App is rebuilt with fixes for items 1-3. Retest only those
   items: verify Sharick name displays without parenthetical, verify
   context menu has 5 items, verify undo toast appears. Update FINDINGS
   status for each. Write recommendation to AGENT OUTPUT LOG.
   Read TEST_STATE.md first for context from previous cycles."]
```

### Writing good NEXT_ACTION prompts

**Be specific.** Not "fix the bugs" — instead "fix the regex in NameParser.swift line 51 that fails on unclosed parentheses from OCR truncation."

**Reference FINDINGS by number.** "Fix items 2 and 5" not "fix the issues."

**State the completion signal.** Always end with "then set STATUS to X" (for Claude Code) or "write recommendation to AGENT OUTPUT LOG" (for Cowork).

**Include verification steps.** For Cowork: "verify the name displays as 'Sharick, Louise M' without '(Lou'." For Claude Code: "build should complete with 0 errors. Warnings OK."

**Include goal context.** Reference the goal ancestry so the agent knows WHY this matters: "Goal context: Mission > Photo Import > Upload Flow."

---

## FINDINGS

> **Cowork writes. Claude Code reads and fixes. Cowork verifies.**
> Each finding has a lifecycle: `open` → `fixed` → `verified` (or back to `open`).

```
[1] severity: major | context_menu_missing_items
    CH09 R4: Right-click context menu missing "Set Description" and
    "Select All in Group". Only shows Assign/Exclude/Transform.
    File: PhotoGridView.swift
    goal: Mission > Photo Organization > Context Actions
    found_by: cowork | status: open

[2] severity: major | parenthetical_name_not_stripped
    CH05: OCR truncated "(Lou" without closing paren not stripped by
    regex in NameParser.swift:51. Regex requires closing ) but OCR
    doesn't always provide it.
    goal: Mission > Photo Import > Name Display
    found_by: cowork | status: open

[3] severity: minor | description_placeholder
    ...
    goal: Mission > Photo Organization > Metadata
    found_by: cowork | status: open
```

### Severity levels

| Level | Meaning | Fix priority |
|-------|---------|-------------|
| `critical` | App crashes, data loss, blocks entire flow | Fix immediately. AUTOPILOT-SAFE freezes on this. |
| `major` | Feature broken or spec violation, but app doesn't crash | Fix in current iteration |
| `minor` | Cosmetic, non-blocking, nice-to-have | Fix if time permits, or defer |

### Finding status lifecycle

```
open ──→ fixed (Claude Code applied fix and rebuilt)
              ──→ verified (Cowork retested and confirmed)
              ──→ open (Cowork retested and it's still broken — count retest failures)

Retest failure tracking:
  status: open | retest_failures: 0   (first time found)
  status: open | retest_failures: 1   (fix didn't work, back to open)
  status: open | retest_failures: 2   (second fix didn't work)
  status: open | retest_failures: 3   → ESCALATE_TO_HUMAN (likely architectural)
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
HEARTBEAT_INTERVAL: 10 minutes (Cowork), fswatch (Claude Code)
TIMEOUT_WINDOW_1:   60 minutes
TIMEOUT_WINDOW_2:   30 minutes
BUDGET_FILE:        FLIGHT_DECK/BUDGET.md
GOAL_FILE:          FLIGHT_DECK/GOAL_ANCESTRY.md
```

### APP BUILD & LAUNCH

**For Xcode projects:** Claude Code builds via `xcodebuild` command line. Cowork does NOT build — it only tests the already-running app.

**APP_BUILD_METHOD:** [xcodebuild CLI | npm run dev | cargo build | etc.]
**APP_LAUNCH:** [open /path/to/App.app | localhost:3000 | etc.]

---

## PATROL PROTOCOL (v2 — Heartbeat-Based)

> All agents follow the unified heartbeat protocol from `HEARTBEAT.md`.
> The protocol below is the FLIGHT_PLAN-specific version — what to do when the heartbeat fires.

### For Claude Code (Brain + Lead Builder)

```
Every heartbeat (fswatch on FLIGHT_PLAN.md):
  1. Read STATUS BLOCK (always first)
  2. Check GLOBAL_STOP — if FROZEN, log and do nothing
  3. Check BUDGET_STATUS — if CAPPED, freeze and write exit summary
  4. Read AGENT OUTPUT LOG for Cowork recommendations
     → If Cowork wrote "Recommend FIX_REQUESTED" → update STATUS to FIX_REQUESTED
     → If Cowork wrote "Recommend RETEST_PASSED" → update STATUS to RETEST_PASSED
  5. Act on STATUS:
     BUILD_REQUESTED or FIX_REQUESTED:
       → Read NEXT_ACTION_FOR_CLAUDE_CODE
       → Read GOAL_CONTEXT for traceability
       → Execute the instructions
       → Create Flight Recorder checkpoint (pre-change)
       → Build and test
       → Log result to AGENT OUTPUT LOG
       → Update STATUS to READY_FOR_TEST
       → Write NEXT_ACTION_FOR_COWORK
       → Update LAST_UPDATED, UPDATED_BY, GOAL_CONTEXT
       → Create Flight Recorder checkpoint (post-change)
     RETEST_PASSED:
       → Check: are all persona/tier combos complete?
       → If yes → set LOOP_COMPLETE, create final checkpoint
       → If no → advance to next persona/tier, set READY_FOR_TEST
     READY_FOR_TEST:
       → Do nothing. Waiting for Cowork.
     LOOP_COMPLETE:
       → Do nothing. Celebration.
  6. Log budget: update BUDGET.md with tokens consumed this heartbeat
  7. Check timeouts:
     → Current task running > TIMEOUT_WINDOW_1 without completion → retry once
     → Retry exceeds TIMEOUT_WINDOW_2 → set ESCALATE_TO_HUMAN
```

### For Cowork Opus (Test Pilot ONLY)

```
Every heartbeat (10-minute timer):
  1. Read STATUS BLOCK (always first)
  2. Check GLOBAL_STOP — if FROZEN, log and do nothing
  3. Check BUDGET_STATUS — if CAPPED, log and do nothing
  4. Act on STATUS:
     READY_FOR_TEST, RETEST_REQUESTED, or AWAITING_TESTER:
       → Read NEXT_ACTION_FOR_COWORK
       → Read TEST_STATE.md for context from previous cycles
       → Read GOAL_CONTEXT to understand what's being tested and why
       → Open the app via computer use
       → Test according to ACTIVE_PERSONA and TEST_FLIGHT_PROTOCOL.md
       → Take screenshots, save to AUDIT_SCREENSHOTS/
       → Write results to FINDINGS (with goal ancestry)
       → Update TEST_STATE.md with progress
       → Write recommendation to AGENT OUTPUT LOG:
         "Recommend FIX_REQUESTED — [N] bugs found. See FINDINGS [1]-[N]."
         OR "Recommend RETEST_PASSED — all items verified."
       → Write NEXT_ACTION_FOR_CLAUDE_CODE with specific fix instructions
       → Update LAST_UPDATED (but NOT STATUS — Claude Code owns that)
     Any other STATUS:
       → Do nothing. Log heartbeat check.
  5. Log budget: note approximate tokens consumed
  6. Check: am I at budget warning (80%) or cap (100%)?
     → 80%: log warning, continue
     → 100%: write exit summary, stop testing
```

### For Second Builder (Optional — Codex, etc.)

```
Every heartbeat (fswatch on FLIGHT_PLAN.md):
  1. Read STATUS BLOCK
  2. Check GLOBAL_STOP, BUDGET
  3. Read AGENT OUTPUT LOG for tasks tagged [CODEX] or [SECOND_BUILDER]
     → If tagged task found → execute it → log results
     → If no tagged task → do nothing
  4. Do NOT modify STATUS BLOCK — only Claude Code (brain) does that
  5. Log budget
```

---

## 🕐 FLIGHT RECORDER INTEGRATION

> See `FLIGHT_RECORDER/README.md` for full Time Machine documentation.

**When Claude Code creates checkpoints:**

| Trigger | Checkpoint Name |
|---------|----------------|
| Stage 1 complete (agreement reached) | `checkpoint_NNN_planning_complete` |
| Before applying fixes to code | `checkpoint_NNN_pre_fix` |
| After successful build, before test | `checkpoint_NNN_pre_test` |
| After all tests pass for a persona/tier | `checkpoint_NNN_test_passed` |
| Before any AUTOPILOT DECISION | `checkpoint_NNN_pre_autopilot` |
| After rollback executed | `checkpoint_NNN_rollback_from_MMM` |
| Human director manually requests | `checkpoint_NNN_manual` |

**To rollback:** Create `FLIGHT_RECORDER/ROLLBACK.md` with target checkpoint. Claude Code reads it on next heartbeat, restores files, and resumes from that checkpoint's stage.

---

## 📊 GOAL ANCESTRY IN FINDINGS

Every finding, every status update, and every NEXT_ACTION should include the goal ancestry line:

```
goal: Mission > [Objective] > [Feature] > [Specific Component]
```

Examples:
```
goal: Mission > Photo Import > Upload Flow > Drag-and-Drop Zone
goal: Mission > Photo Organization > Context Menu > Set Description Action
goal: Mission > Photo Viewing > Grid View > Thumbnail Rendering
```

This ensures the human director can see at a glance which objectives are validated and which are blocked — not just which buttons passed or failed. See `GOAL_ANCESTRY.md` for the full mission tree.

---

## AGENT OUTPUT LOG

> **Append-only. Both agents write here. Neither edits previous entries.**
> This is the immutable record of everything that happened.

```
[YYYY-MM-DD HH:MM] — [Agent Name]
Action: [what was done]
Result: [outcome]
Budget: [tokens consumed this cycle]
Goal: [goal ancestry context]

[YYYY-MM-DD HH:MM] — Cowork Opus
Action: Tested upload flow as Cold User (Pat)
Result: 3 bugs found — see FINDINGS [1]-[3]. Recommend FIX_REQUESTED.
Budget: ~18,000 tokens
Goal: Mission > Photo Import > Upload Flow

[YYYY-MM-DD HH:MM] — Claude Code
Action: Fixed FINDINGS [1]-[3]. Rebuilt. App launched.
Result: Build successful (0 errors, 2 warnings). STATUS → READY_FOR_TEST.
Budget: ~4,500 tokens
Goal: Mission > Photo Import > Upload Flow
Checkpoint: checkpoint_004_pre_test created

[YYYY-MM-DD HH:MM] — Claude Code [AUTOPILOT DECISION]
Action: Critical bug in upload — chose safest fix (disable drag-and-drop, keep file picker).
Result: Partial fix. File picker works. Drag-and-drop deferred to next iteration.
Budget: ~6,200 tokens
Goal: Mission > Photo Import > Upload Flow
Checkpoint: checkpoint_005_pre_autopilot created
```

---

## ACTIVE PERSONA / TIER TRACKING

```
ACTIVE_PERSONA:     [Alex | Jordan | Pat | Sam]
ACTIVE_TIER:        [Cold | Guided | Insider]
TIERS_COMPLETED:    [e.g., "Cold: PASS, Guided: in progress, Insider: pending"]
TIER_GAP_DATA:      [e.g., "Cold: 9 steps, Guided: 5 steps, Insider: 3 steps"]
TIER_GAP_RATIO:     [e.g., "3.0 (Cold/Insider) — target: < 2.0"]
```

---

## ESCALATION TRIGGERS

These apply to ALL agents, checked every heartbeat:

| Condition | Action | Who Acts |
|-----------|--------|----------|
| `GLOBAL_STOP: FROZEN` | Halt immediately, log state | All agents |
| Budget at 80% | Log warning to FLIGHT_LOG.md, continue | The capped agent |
| Budget at 100% | Stop, write exit summary, freeze | The capped agent + Claude Code freezes |
| Build fails 2x on same fix | Escalate to human before third attempt | Claude Code |
| Bug fails retest 3x | Escalate to human — likely architectural | Claude Code |
| S1 Blocker found during testing | Alert human + pause immediately | Cowork recommends, Claude Code freezes |
| Confidence < 60% on any action | Alert human before proceeding | Claude Code |
| No file changes for 30+ min (active stage) | Write staleness warning | Claude Code |
| Cowork stuck 30+ min with no update | Write escalation to FLIGHT_PLAN.md | Claude Code |
| Flight Planning: 3+ rounds with no new agreement | Escalate — agents may be deadlocked | Claude Code |

---

## CHANGELOG

```
v2.0.0 — March 2026
  - Claude Code is now the brain (was Cowork in v1)
  - Cowork is pure test pilot only (was brain + tester in v1)
  - Added three-stage architecture (Flight Planning → Flight → Test Flight)
  - Added unified heartbeat protocol (HEARTBEAT.md)
  - Added budget circuit breakers (BUDGET.md)
  - Added Flight Recorder / Time Machine (FLIGHT_RECORDER/)
  - Added goal ancestry chain (GOAL_ANCESTRY.md)
  - Added persistent test state (TEST_STATE.md)
  - Added governance gates for stage transitions
  - Cowork writes recommendations, Claude Code owns STATUS transitions
  - FLIGHT_LOG.md now append-only / immutable
  - Inspired by Paperclip (github.com/paperclipai/paperclip) orchestration patterns

v0.2.0 — Original
  - Dual-Patrol architecture
  - Cowork as brain + test pilot
  - 10-minute polling for both agents
  - fswatch as optional acceleration
```
