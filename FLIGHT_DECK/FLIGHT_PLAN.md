# 🧠 Flight Plan — Multi-Agent Orchestration Protocol v0.1.0
**Cowork (Opus) is THE BRAIN — patrol, testing, and decisions all in one model.**
**Now with: Ralph Wiggum Build Loop · Test Pilot Validation Gate · Kill Switch · Confidence Scores · Opus 10-Min Patrol**

---

## 🏗️ ARCHITECTURE OVERVIEW

```
┌─────────────────────────────────────────────────────────────────┐
│                     HUMAN DIRECTOR (You)                        │
│              Triggers Cowork manually or via 10-min patrol               │
└──────────────────────────┬──────────────────────────────────────┘
                           │
┌──────────────────────────▼──────────────────────────────────────┐
│              🟣 COWORK — OPUS (THE BRAIN + TEST PILOT)          │
│                                                                 │
│   • Controls Ralph Wiggum build loops (writes prompts)          │
│   • Runs Test Pilot validation after each build cycle           │
│   • Feeds UX findings back into Ralph for next iteration        │
│   • Reads/writes Obsidian vault for persistent memory           │
│   • Makes ALL architectural and design decisions                │
│   • WAKES via: 10-min patrol trigger OR human manual trigger          │
│                                                                 │
│   THE COMBINED ENGINE:                                          │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │ 🔁 RALPH WIGGUM LOOP (Build) → 🧪 TEST PILOT (Validate)│   │
│   │    Code passes? ──NO──→ Ralph iterates                  │   │
│   │    UX passes?   ──NO──→ findings fed back → Ralph again │   │
│   │    BOTH pass?   ──YES─→ ✅ DONE                        │   │
│   └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│   COMMUNICATION CHANNELS:                                       │
│   📝 Write feedback to FLIGHT_PLAN.md (shared file)             │
│   🖥️  Then Cmd+Tab to Terminal → type instructions to code agent │
│                                                                 │
└──┬──────────┬──────────┬──────────┬─────────────────────────────┘
   │ .md+term │ .md+term │ .md      │ .md
   ▼          ▼          ▼          ▼
┌──────┐ ┌──────┐ ┌──────────┐ ┌──────────┐
│🔵 CC │ │🟢 B2 │ │🟡 Patrol │ │🧪 Test  │
│Build │ │Build │ │ Check    │ │ Pilot   │
│Ralph │ │Ralph │ │ ALL agts │ │ App Test│
│Loop  │ │Loop  │ │ 10-min   │ │ UX+Func │
└──────┘ └──────┘ └──────────┘ └──────────┘
```

### 📡 Communication Channel Details

**📝 PRIMARY — Shared .md File (Normal Operations)**
Cowork writes all feedback, task assignments, and direction into `FLIGHT_PLAN.md`. Agents are responsible for reading the file before every action. This is the default channel for all routine communication.

**🖥️ FALLBACK — Direct Terminal Control (Override)**
When the .md channel fails — agent is stuck, not reading the file, hung process, or urgent intervention needed — Cowork uses computer use to:
1. Switch to the agent's terminal window/tab
2. Type instructions directly into the Claude Code or Second Builder CLI session
3. Issue commands: interrupt current work, redirect, restart, or provide context

**When to use fallback:**
- Agent has been `⏳ Awaiting Review` but hasn't picked up feedback after file update
- Agent appears stuck in a loop or producing incorrect output
- Urgent redirect needed mid-execution (e.g., "STOP — wrong branch")
- Need to pass context that's too complex for the .md file (e.g., paste error logs)

**Fallback logging:** When Cowork uses direct terminal control, it must log the intervention in the AGENT OUTPUT LOG:
```
[YYYY-MM-DD HH:MM] — Opus DIRECT TERMINAL INTERVENTION
Target Agent: [Claude Code / Second Builder]
Reason: [why .md channel was insufficient]
Action Taken: [what was typed/commanded]
Result: [agent response]
```

---

## 🧠 SECOND BRAIN — OBSIDIAN VAULT

> Cowork's project memory is session-scoped and volatile.
> A persistent knowledge base (e.g., an Obsidian vault, a shared folder of .md files, or any durable notes system) is the **durable memory** that persists across all sessions.

### Vault Structure (Example)
```
📁 YourProject-SecondBrain/            (Obsidian vault root)
├── 📁 _Orchestration/
│   ├── decisions-log.md               ← Every Opus decision with reasoning
│   ├── architecture-decisions.md      ← ADRs (Architecture Decision Records)
│   ├── lessons-learned.md             ← What went wrong, what worked
│   └── agent-performance.md           ← Which agent excels at what
├── 📁 ProjectA/
│   ├── current-state.md               ← Latest build status, known issues
│   ├── technical-debt.md              ← Deferred fixes, TODOs
│   └── session-handoffs.md            ← Context for resuming work
├── 📁 ProjectB/
│   ├── current-state.md
│   ├── technical-debt.md
│   └── session-handoffs.md
└── 📁 ProjectC/
    ├── current-state.md
    ├── technical-debt.md
    └── session-handoffs.md
```

### Second Brain Protocol
1. **Session Start**: Cowork reads `_Orchestration/` + relevant project `current-state.md` to restore full context
2. **During Session**: Cowork writes decisions and findings in real time to the vault
3. **Session End**: Cowork writes a `session-handoffs.md` entry summarizing: what was done, what's next, any blockers, and open questions
4. **Cross-Session Continuity**: Any agent spawned by Cowork receives relevant vault context in its spawn prompt

### Cowork Vault Access
Cowork accesses the Obsidian vault by **reading and writing .md files directly** in the vault folder via computer use. No MCP plugin required — Cowork opens, reads, and edits files natively through the desktop environment.

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
5. ✅ Claude Code / Second Builder sessions can read/write the project folder
6. ✅ AUDIT_SCREENSHOTS/ folder exists

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
> ⚡ Opus checks this section every 10 minutes during patrol mode. All agents check this before starting work.
> 🛑 ALL agents check GLOBAL STOP before EVERY action. If FROZEN → do nothing.
> ✈️ ALL agents check PILOT MODE to know whether the human director is available for escalation.

```
GLOBAL STOP     : [ACTIVE / 🛑 FROZEN]
PILOT MODE      : [👨‍✈️ SUPERVISED / ✈️ AUTOPILOT / 🛬 AUTOPILOT-SAFE]
PROJECT         : [Your Project Name]
LAST UPDATED    : [YYYY-MM-DD HH:MM]
CURRENT PHASE   : [Planning | Building | Auditing | Testing | Review | Complete]
ACTIVE AGENTS   : [Claude Code | Second Builder | Cowork Opus]
BLOCKING ISSUE  : [None | Describe]
ESCALATE OPUS   : [YES | NO]
WAKE REASON     : [Opus patrol trigger | Human manual | Scheduled review]
NEXT PATROL CHECK: [YYYY-MM-DD HH:MM]
AUTOPILOT UNTIL : [YYYY-MM-DD HH:MM / Indefinite / N/A]
```

### 🛑 KILL SWITCH — GLOBAL STOP
**Any agent or the human director can set `GLOBAL STOP: 🛑 FROZEN` at any time.**

When FROZEN:
- ALL agents immediately stop current work
- No new tasks may be started
- No auto-proceeding
- Agents write their current state to AGENT OUTPUT LOG with status `🛑 Frozen`
- Opus patrol continues but takes NO routing actions — only logs status
- Only Cowork Opus or the human director can set `GLOBAL STOP: ACTIVE` to resume

When to freeze:
- System going in wrong architectural direction
- Agents caught in infinite loop or cascading errors
- Need to rethink approach before more work is wasted
- Any agent detects it's producing output that conflicts with PRD
- The human director needs to step away and doesn't want agents running unsupervised (use this OR autopilot)

### ✈️ PILOT MODE — Human Supervision Level

The human director sets this before stepping away. It controls how much autonomy Cowork Opus has.

#### 👨‍✈️ SUPERVISED (Default)
Human director is available. Normal operations.
- Cowork Opus escalates to the human director per the escalation matrix
- Human director approves S1/S2 critical bugs, design decisions, priority conflicts
- Full governance — nothing ships without human awareness

#### ✈️ AUTOPILOT
Human director is away. Cowork Opus has full authority.
- Cowork Opus makes ALL decisions that would normally escalate to the human director
- Opus handles S1/S2 bugs on its own — chooses the safest fix approach
- Opus makes design/UX decisions based on PRD and existing patterns
- Opus can approve, reject, reassign, and restructure anything
- **Guardrails still active:**
  - Kill switch can still fire (any agent can freeze the system)
  - Confidence < 60% on any decision → Opus logs it but HOLDS the task for the human director's return instead of deciding
  - No deployment to production / no irreversible actions
  - All decisions logged to Obsidian vault with `[AUTOPILOT DECISION]` tag so the human director can review later
  - Opus 10-minute patrol continues — monitoring doesn't stop
- **When autopilot expires** (reaches `AUTOPILOT UNTIL` time):
  - Mode automatically reverts to 👨‍✈️ SUPERVISED
  - Any held tasks are queued for human review
  - Opus writes a summary: "Autopilot session complete. X tasks completed, Y decisions made, Z items held for your review."

#### 🛬 AUTOPILOT-SAFE (Conservative Autopilot)
Human director is away but wants minimal risk.
- Cowork Opus can only proceed with 🟢 Low and ⚪ Trivial tasks
- ALL 🟡 Medium and 🔴 High tasks are HELD until the human director returns
- S1/S2 bugs → Opus sets `GLOBAL STOP: 🛑 FROZEN` and waits for the human director
- S3-S5 bugs → Opus sends to build agent, fixes proceed normally
- No architectural decisions, no design changes, no new task creation
- Essentially: "keep doing easy work, stop if anything hard comes up"

### How the Human Director Activates Autopilot

**Before stepping away:**
```
Human director sets in STATUS BLOCK:
  PILOT MODE: ✈️ AUTOPILOT
  AUTOPILOT UNTIL: 2026-03-27 14:00

Or conservative:
  PILOT MODE: 🛬 AUTOPILOT-SAFE
  AUTOPILOT UNTIL: 2026-03-27 14:00
```

**When returning:**
```
Human director sets:
  PILOT MODE: 👨‍✈️ SUPERVISED
  AUTOPILOT UNTIL: N/A
```
Then reads the autopilot summary in the OPUS DECISION LOG and reviews any held items.

### Autopilot Decision Logging

Every decision Opus makes in autopilot mode gets a special tag in the Obsidian vault:

```
[YYYY-MM-DD HH:MM] — [AUTOPILOT DECISION]
Mode: ✈️ AUTOPILOT / 🛬 AUTOPILOT-SAFE
Decision: [what was decided]
Normally would escalate because: [reason this would go to the human director]
Opus reasoning: [why Opus chose this approach]
Confidence: [0-100%]
Reversible: [YES — can be undone / NO — permanent]
Held for human review: [YES — Opus wasn't confident enough / NO — Opus proceeded]
```

### Autopilot Summary (Written When Mode Ends)

```
[YYYY-MM-DD HH:MM] — AUTOPILOT SESSION SUMMARY
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Duration: [start time] → [end time]
Mode: ✈️ AUTOPILOT / 🛬 AUTOPILOT-SAFE

Tasks completed: [count]
Tasks held for human review: [count]
Bugs found: [count by severity]
Bugs fixed: [count]
Test pilots performed: [count]
Decisions made that would normally escalate: [count]

KEY DECISIONS (review these):
  1. [Decision — e.g., "Chose approach B for BUG-T004-001 (mesh rendering)"]
  2. [Decision — e.g., "Approved Second Builder plan for T008 without human review"]
  3. [Decision — e.g., "Held T009 — confidence 55%, needs your input"]

ITEMS REQUIRING YOUR REVIEW:
  • T009 — [description, why it was held]
  • BUG-T002-005 — [S2 Critical, Opus fixed it but wants your validation]

OVERALL HEALTH:
  System stable: YES / NO
  Any kill switch triggers: [none / describe]
  Dimension score trend: [improving / stable / declining]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## 📋 PROJECT BRIEF

- **Project:**
- **PRD Location:** `[path/to/PRD.md]`
- **Goal:**
- **Platform:** [iOS / macOS / Windows PC / Remote Desktop]
- **Obsidian Vault:** `[path/to/YourProject-SecondBrain/]`
- **Mounted Folders:** [list of mounted paths]
- **Success Criteria:**

---

## 🎯 TASK QUEUE
> **Only Cowork Opus updates this table.** Opus updates Status during patrol checks.
> Agents MUST NOT self-assign or start tasks not listed here.

| ID   | Task | Assigned To | Status             | Risk Level | Depends On | Notes |
|------|------|-------------|--------------------|------------|------------|-------|
| T001 |      | Claude Code | 🔲 Pending         | 🟢 Low     |            |       |
| T002 |      | Second Builder       | 🔲 Pending         | 🟡 Medium  |            |       |
| T003 |      | Cowork Test | 🔲 Pending         | —          |            |       |

**Status keys:**
- 🔲 Pending — Not started, waiting for assignment
- 📋 Planning — Agent is writing plan (Second Builder default entry state)
- 🔄 In Progress — Actively building/executing
- 🟢 Auto-Proceeding — Claude Code auto-advancing through low-risk sequential tasks
- ⏳ Awaiting Review — **WORK COMPLETE. AGENT STOPPED. Waiting for Cowork Opus feedback.**
- 🧪 Test-Piloting — Cowork Opus is currently testing this feature live
- ✅ Approved — Cowork Opus reviewed AND test-piloted successfully
- 🔁 Revision Needed — Cowork Opus reviewed/tested and sent back with comments
- ❌ Blocked — Cannot proceed, needs escalation
- 🔍 Under Review — Cowork Opus is currently reviewing code/output

**Risk levels (set by Cowork Opus when creating tasks):**
- ⚪ Trivial — Comment fixes, import ordering, logging changes, typos, formatting. **Opus fast-approves during patrol — no deep test needed.**
- 🟢 Low — Routine, no architectural impact, single file, well-defined. Claude Code may auto-proceed. Opus batch-reviews.
- 🟡 Medium — Multiple files, some design judgment needed. Requires Opus review + test pilot before next task.
- 🔴 High — Architectural change, new patterns, cross-module impact. Full stop + Opus review + test pilot.

### ⚪ PATROL FAST-APPROVE (Trivial Tasks)
For ⚪ Trivial tasks, Opus fast-approves during patrol check — no full test mode needed:
1. Claude Code completes ⚪ task → writes output → sets `⏳ Awaiting Review`
2. Opus sees it on next 10-minute patrol check
3. Opus quick-reviews: Did the change match the task description? Any errors? Only expected files modified?
4. If clean → Opus sets status `✅ Approved (patrol fast-approve)` → CC continues
5. If anything unexpected → Opus switches to full test mode
6. Opus logs fast-approvals in patrol log

**What qualifies as ⚪ Trivial (Opus sets this — agents cannot self-assign):**
- Adding/removing comments or print statements
- Import reordering or unused import cleanup
- Variable renaming (within a single file)
- Formatting / linting fixes
- Updating string literals (labels, error messages)
- Adding logging statements

**What is NOT ⚪ Trivial (even if it seems small):**
- Any logic change, even one line
- Any new file creation
- Any API or interface change
- Any change to a file that another agent is also modifying

---

## 🤖 AGENT ROLES & RULES

### 🟣 Cowork — Opus (THE BRAIN + TEST PILOT)
**The single source of authority for all project decisions. Also the primary test pilot.**
- Wakes when `ESCALATE OPUS: YES` OR when human triggers manually
- On wake: reads STATUS BLOCK → reads full AGENT OUTPUT LOG → reads relevant Obsidian vault notes
- Decomposes high-level goals into task queue entries with **risk levels**
- Reviews ALL agent output before approving next steps
- **Writes feedback/direction/tasks into `FLIGHT_PLAN.md`** — the shared record
- **Then switches to Terminal via computer use** and types instructions directly into the Claude Code / Second Builder session, telling the code agent to read the feedback and act on it
- After deciding: updates task queue, writes feedback, sets `ESCALATE OPUS: NO`
- Writes key decisions to Obsidian vault `_Orchestration/decisions-log.md`
- Logs all direct terminal interventions in AGENT OUTPUT LOG
- **TEST PILOT:** After approving any 🟡 Medium or 🔴 High risk task, Cowork Opus test-pilots the app through computer use — clicking, typing, and navigating (see TEST PILOT PROTOCOL below)
- **ONLY agent that can**: restructure task queue, reassign tasks, change architecture, resolve conflicts, directly control other agent terminals, set risk levels

### 🟡 Opus Patrol Mode (10-Minute Check — ALL Agents + Fast-Approver)
- **Checks in every 10 minutes** — reads STATUS BLOCK + scans AGENT OUTPUT LOG for changes
- **FIRST CHECK: Is GLOBAL STOP set to 🛑 FROZEN?** If yes → log patrol, take NO actions, wait for unfreeze.
- **Patrols ALL agents on every check-in:**
  - **Claude Code status check:** Is CC building? Stuck? Auto-proceeding? Erroring? How many tasks completed since last check? What are the confidence scores?
  - **Second Builder status check:** Is Second Builder waiting for plan approval? Executing? How long has it been in current state?
  - **Cowork Opus Test Status:** Is the tester running? Any new findings? Screenshots captured?
  - **Cowork Opus:** Is Opus awake? When did it last write feedback? Is it test-driving?
- **Fast-approve authority (⚪ Trivial tasks only):**
  - If a ⚪ Trivial task is `⏳ Awaiting Review` → Opus reviews it in patrol mode
  - Opus checks: correct files modified? No logic changes? No errors? Matches task description?
  - If clean → Opus approves directly: `✅ Approved (patrol fast-approve)`
  - If anything unexpected → Opus switches to full test mode
  - **Opus patrol NEVER fast-approves anything above ⚪ Trivial**
- **Confidence monitoring:**
  - If any agent reports confidence < 80% → Opus switches to full test mode immediately
  - If CC auto-proceeded with confidence 80-90% for 3+ tasks → Opus flags for full review
- **Routing actions on each check-in:**
  - If Second Builder has a new task assigned → Opus initiates Second Builder with: **"Plan mode only. Write your plan to FLIGHT_PLAN.md under your Agent Output Log entry. Do NOT execute. Wait for Cowork Opus approval."**
  - If Claude Code has a new task assigned → Opus passes the task with risk level and auto-proceed rules
  - If any agent is `⏳ Awaiting Review` (non-trivial) → Opus enters full review mode
  - If Claude Code has been `🟢 Auto-Proceeding` for 3+ tasks without Opus check → Opus flags for batch review
  - If any agent has been in same status for >10 min → Opus investigates and logs finding
- Grants routine permission approvals (file access, non-architectural decisions)
- Escalates to Opus if: blocked >10 min, 3+ audit failures, architecture decision needed, agent conflict
- Appends check-in to PATROL CHECK-IN LOG

### 🔵 Claude Code (Builder + Code Auditor — With Autopilot)
- Default model: Sonnet. Switch to Opus for complex decisions.
- **MUST write a plan before any action**
- Executes assigned tasks from the task queue

**Autopilot mode (🟢 Low risk tasks):**
- When a task is marked 🟢 Low AND the next task in queue is also 🟢 Low with no dependency:
  - CC completes task → writes output to AGENT OUTPUT LOG → sets status to `🟢 Auto-Proceeding`
  - CC **immediately starts the next 🟢 Low task** without waiting for Opus
  - CC still logs every task completion — Opus can review in batches
  - If CC encounters ANY error, unexpected result, or uncertainty → **immediately stops, sets ⏳ Awaiting Review**
  - **If CC's confidence score drops below 80% → immediately stops, sets ⏳ Awaiting Review**
  - Opus patrol checks on CC every 10 minutes and flags for Opus batch review after 3+ auto-proceeded tasks

**Trivial mode (⚪ Trivial tasks):**
- CC completes task → writes output → sets `⏳ Awaiting Review`
- **Opus fast-approves on next patrol check** — Opus is NOT woken
- CC may auto-proceed to next ⚪ task if Opus has already fast-approved the pattern

**Standard mode (🟡 Medium / 🔴 High risk tasks):**
- CC completes task → writes output to AGENT OUTPUT LOG → sets status to `⏳ Awaiting Review` → **STOPS**
- **DOES NOT start next task until Cowork Opus writes feedback/approval in FEEDBACK QUEUE**

- Checks FEEDBACK QUEUE for direction before every new action
- Flags blockers immediately in STATUS BLOCK

### 🟢 Second Builder (Builder — Plan First, Always)
- **ALWAYS enters in plan-only mode when initiated by Opus patrol**
- Writes detailed plan to AGENT OUTPUT LOG → sets status to `📋 Planning` → **STOPS**
- **DOES NOT execute until Cowork Opus explicitly approves the plan**
- After plan approval: executes → writes output → sets status to `⏳ Awaiting Review` → **STOPS**
- **Second Builder NEVER auto-proceeds.** Every task requires full plan → approve → execute → review cycle.
- Append only — never overwrite other agent entries
- Checks FEEDBACK QUEUE for direction before every new action

### 🟠 Opus Test Mode (Live App Testing — Triggered by Patrol)
- Uses computer use to interact with the running app
- Tests all features against PRD expected outcomes
- Captures screenshots as evidence → saves to `AUDIT_SCREENSHOTS/`
- Reports UX feedback: Smooth ✅ · Clunky ⚠️ · Broken ❌
- **Runs continuously during build phases** — tests each feature as soon as it's built, not just at completion
- Writes findings to AUDIT LOG → sets relevant task to `⏳ Awaiting Review` if issues found
- Triggered by patrol mode when "Ready for test" appears — reports status on each check-in

---

## 🧪 TEST PILOT PROTOCOL — COWORK OPUS AS REAL HUMAN TESTER
> After every completed step (🟡 Medium or 🔴 High), Cowork Opus becomes the test pilot.
> This is NOT automated testing. This is Opus using the app like a real person would.

### What "Test Driving" Means
Cowork Opus uses computer use to:
1. **Launch the app** — open Xcode simulator, run the build, or open the project in its target environment
2. **Interact through computer use** — tap buttons, scroll, navigate between screens, enter data, try the feature that was just built
3. **Try edge cases** — what happens with empty input? With very long text? With rapid tapping? Rotating the device?
4. **Follow the user journey** — don't just test the new feature in isolation. Walk through the full flow a real user would take: open app → create project → use the feature → review results
5. **Look at it visually** — is the UI aligned? Do colors look right? Does the layout break on different content?
6. **Try to break it** — a real QA human tries to find bugs, not confirm the happy path

### Test Pilot Workflow
```
1.  Agent completes 🟡/🔴 task → sets ⏳ Awaiting Review
2.  Opus reviews code/output in AGENT OUTPUT LOG
3.  Opus sets task status → 🧪 Test-Piloting
4.  Opus launches the app via computer use
5.  Opus interacts with the app through computer use:
    ┌──────────────────────────────────────────────┐
    │  • Open the app / navigate to the feature    │
    │  • Try the happy path (expected use)          │
    │  • Try edge cases (empty, long, rapid, wrong) │
    │  • Check visual alignment and layout          │
    │  • Test the full user journey, not just the   │
    │    isolated feature                           │
    │  • Take screenshots of anything notable       │
    └──────────────────────────────────────────────┘
6.  Opus writes TEST PILOT REPORT (see below)
7.  If PASS → sets status ✅ Approved → writes feedback → next task
8.  If FAIL → sets status 🔁 Revision Needed → writes specific bugs found → agent fixes
```

### Test Pilot Report Template
```
[YYYY-MM-DD HH:MM] — Opus Test Pilot Report
Task ID: T001
Feature Tested: [what was built]
App State: [how was the app launched — simulator, build, remote desktop]

Happy Path:
  • [step tried] → [result] ✅/❌
  • [step tried] → [result] ✅/❌

Edge Cases:
  • Empty input → [result] ✅/❌
  • Long text → [result] ✅/❌
  • Rapid interaction → [result] ✅/❌
  • [other edge case] → [result] ✅/❌

Visual Check:
  • Layout: [aligned / broken / minor issues]
  • Colors/fonts: [correct / off]
  • Responsive to content: [yes / no]

User Journey Test:
  • Full flow from [start] to [end]: [completed / broke at step X]

Screenshots: [paths to saved screenshots]

Verdict: ✅ PASS — Ready for next task / ❌ FAIL — Needs revision
Bugs Found:
  • [specific bug 1 — what happened, where, steps to reproduce]
  • [specific bug 2]
Feedback to Agent:
  [specific direction for fixes]
```

### When to Test Pilot vs. Batch Review
- 🔴 High risk → **Always test pilot immediately**
- 🟡 Medium risk → **Test pilot after each task**
- 🟢 Low risk (auto-proceed) → **Opus batch-reviews code only.** Opus handles app testing for low-risk items in patrol mode. Opus test-pilots a sample of 🟢 tasks every ~3 batch reviews.

---

## 📬 FEEDBACK QUEUE
> **Cowork Opus writes here. Agents read before every action.**
> This is the primary direction channel — agents must check this before starting or resuming work.

```
[YYYY-MM-DD HH:MM] — Opus → Claude Code
Re: T001
Decision: APPROVED / REVISION NEEDED / HOLD
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
7.  Switch to Terminal (Cmd+Tab) → type into Claude Code session:
    • "Read FLIGHT_PLAN.md — test pilot feedback for T001. Fix the
       issues, rebuild, relaunch, and update file when ready for retest."
    • This directly tells the code agent to act on the feedback
    • No waiting for the agent to discover the file change on its own
8.  Write decisions to Obsidian vault (decisions-log.md)
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
| Agent not responding to .md feedback     | Opus uses FALLBACK terminal control  |
| Agent stuck in loop / wrong direction    | Opus uses FALLBACK terminal control  |
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

### Direct Terminal Control (Fallback)
> Only Cowork Opus may use this channel. All other agents communicate exclusively via the .md file.

8. Cowork Opus may switch to an agent's terminal via computer use when:
   - Agent has not responded to .md feedback within ~2 minutes
   - Agent is stuck in a loop or producing incorrect output
   - Urgent mid-execution redirect is needed ("STOP — wrong approach")
   - Complex context (error logs, stack traces) needs to be pasted directly
9. When using terminal fallback, Cowork types clearly prefixed instructions:
   ```
   [OPUS DIRECT] Read FLIGHT_PLAN.md now — new feedback for T001
   [OPUS DIRECT] STOP current work. Architectural change required. See FEEDBACK QUEUE.
   [OPUS DIRECT] Context for your current task: [paste relevant info]
   ```
10. Every terminal intervention MUST be logged in the AGENT OUTPUT LOG with:
    - Timestamp, target agent, reason for fallback, action taken, result
11. After terminal intervention, resume using the .md file as primary channel

---

## 🔄 CRASH RECOVERY PROTOCOL
> What to do when a session dies, an agent crashes, or context is lost.

### If Cowork Opus session crashes:
1. Human director manually restarts Cowork Opus
2. Opus reads Obsidian vault `_Orchestration/decisions-log.md` + project `session-handoffs.md` to restore context
3. Opus reads FLIGHT_PLAN.md STATUS BLOCK + AGENT OUTPUT LOG to see current state
4. Opus resumes from last known state — re-reviews any `⏳ Awaiting Review` items
5. Opus writes: `[YYYY-MM-DD HH:MM] — Opus RECOVERED from session crash. Context restored from Obsidian vault.`

### If Claude Code session crashes:
1. Opus detects on next 10-minute patrol (status unchanged, no output for >10 min)
2. Opus enters full review mode with note: "Claude Code appears crashed"
3. Opus restarts Claude Code via terminal (fallback channel) or notifies the human director
4. New CC session reads CLAUDE.md (which imports AUTOPILOT.md) + FLIGHT_PLAN.md to restore context
5. CC resumes from last logged task — any in-progress work is re-executed from the plan

### If Second Builder session crashes:
1. Opus detects on next 10-minute patrol
2. Opus re-initiates Second Builder in plan-only mode for the current task
3. If Second Builder had an approved plan, Opus passes: "Your plan was already approved. Resume execution from step [X]."
4. If Second Builder was still planning, Opus passes: "Restart your plan for task T00X."

### If Opus patrol stops (session timeout or crash):
1. Cowork Opus notices no patrol entries for >10 minutes
2. Human director notices patrol has stopped (no recent patrol entries).
3. Human director restarts Cowork with Opus and re-gives patrol instruction.
4. Opus resumes patrol from where it left off.

### Recovery state file:
After any crash recovery, Opus writes to Obsidian vault `_Orchestration/lessons-learned.md`:
```
[YYYY-MM-DD HH:MM] — Crash Recovery
What crashed: [agent name]
What was lost: [any work that couldn't be recovered]
How recovered: [steps taken]
Prevention: [what to do differently next time]
```

---

## 📁 FILE REFERENCES

| File                         | Purpose                                               |
|------------------------------|-------------------------------------------------------|
| `PRD.md`                     | Product requirements — single source of truth          |
| `FLIGHT_PLAN.md`     | This file — shared communication hub                   |
| `CLAUDE.md` + `FLIGHT_DECK/AUTOPILOT.md` | Claude Code project-level instructions  |
| `AUDIT_SCREENSHOTS/`         | Folder for Cowork test screenshots + test pilot screenshots |
| `YourProject-SecondBrain/`   | Obsidian vault — persistent memory across sessions     |

### 🔮 FILE-SPLIT ROADMAP (When Contention Gets Real)
> **Don't split now.** Run v0.1.0 as a single file first. Split ONLY when you see actual write collisions.

When the single `FLIGHT_PLAN.md` file starts causing problems (two agents writing simultaneously, corrupted entries, logs getting too long to read), split into:

| File | What moves there | Who writes | Who reads |
|------|-------------------|------------|-----------|
| `TASK_QUEUE.md` | Task queue table only | Opus only | Everyone |
| `FEEDBACK.md` | Feedback queue entries | Opus only | CC, Second Builder |
| `AGENT_LOG.md` | Agent output log entries | CC, Second Builder, Testers | Opus |
| `STATUS.md` | Status block + kill switch | Opus (status), Anyone (kill switch) | Everyone |
| `PATROL_LOG.md` | Opus patrol entries | Opus only | Human director |
| `AUDIT_LOG.md` | Audit + test pilot reports | Opus, Testers | Opus, CC |

**Split trigger:** If you see `[AGENT NAME — WRITING]` collisions in logs more than twice in one session, it's time. Until then, one file is simpler.

---

## 🔑 CRITICAL RULES — ALL AGENTS READ THIS

0. **CHECK GLOBAL STOP FIRST.** Before every action, check `GLOBAL STOP`. If `🛑 FROZEN` → do nothing. Log your state and wait.
1. **CHECK PILOT MODE.** Determines whether to escalate to the human director or let Opus decide autonomously.
   - 👨‍✈️ SUPERVISED = Escalate to the human director per the escalation matrix
   - ✈️ AUTOPILOT = Opus decides everything. Human director reviews when they return.
   - 🛬 AUTOPILOT-SAFE = Only 🟢/⚪ tasks proceed. Everything else holds.
2. **Cowork Opus is the ONLY decision-maker.** No agent makes architectural, design, or strategic decisions independently. In autopilot, Opus assumes the human director's authority within guardrails.
3. **Risk level determines workflow:**
   - ⚪ Trivial = Opus fast-approves during patrol, no full test needed
   - 🟢 Low = CC can auto-proceed, Opus batch-reviews
   - 🟡 Medium / 🔴 High = full stop + Opus review + test pilot
4. **Second Builder NEVER auto-proceeds.** Always: plan → stop → approval → execute → stop → review.
5. **Claude Code checks risk level before deciding to auto-proceed or stop.** When in doubt, stop.
6. **Report confidence scores honestly.** If below 80%, stop and escalate. Don't auto-proceed with low confidence.
7. **Check FEEDBACK QUEUE before every action.** If Opus has written feedback for you, follow it before doing anything else.
8. **Never start a task not in the Task Queue.** If you think a task is needed, flag it in your output and let Opus decide.
9. **Append only.** Never delete or overwrite another agent's entries in any log section.
10. **When in doubt, escalate.** Set `ESCALATE OPUS: YES` rather than making an uncertain decision.
11. **Opus patrols every 10 minutes.** ALL agents are monitored. No agent operates unwatched — even in autopilot.
12. **Opus test-pilots after every 🟡/🔴 task.** The app must be usable through computer use before a task is approved.
13. **Anyone can pull the kill switch.** If the system is going wrong, set `GLOBAL STOP: 🛑 FROZEN`. This overrides ALL modes including autopilot.
14. **Autopilot is not unsupervised.** Opus still applies full governance. Autopilot means Opus makes the calls the human director would make — not that governance is relaxed. Every autopilot decision is logged with reasoning and confidence for the human director to review later.

---

*Flight Plan v0.1.0 — Test Pilot Loop Framework by Huan Su*
*Ralph Wiggum Loop (Build) + Test Pilot Loop (Validate) = Ship with confidence*
*Cowork Opus = Brain + Patrol + Test Pilot · Ralph = Build Engine · Shared File = Communication*
