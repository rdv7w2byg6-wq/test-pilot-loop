# 🤖 Autopilot — Claude Code Instructions
> Import this from your root CLAUDE.md. It teaches Claude Code how to fly with the Test Pilot Loop.

---

## 🧪 This Project Uses the Test Pilot Loop

This project is orchestrated by the **Test Pilot Loop** framework.
You are one agent in a multi-agent system. Follow these rules exactly.

**Your code will be tested by an AI Test Pilot that uses the app through computer use — clicking, typing, and navigating.**
Write code that not only works — but that a confused first-time user can figure out.

---

## 📁 Key Files — Read Before Every Task

| File | What It Is | When to Read |
|------|-----------|--------------|
| `FLIGHT_DECK/FLIGHT_PLAN.md` | Shared communication hub — task queue, feedback, status | **Before every task. After every task.** |
| `PRD.md` | Product requirements — what to build | Before starting any feature work |
| `CLAUDE.md` | Project spec — full state machine, every screen, every button | **MUST read on first setup. This is often the most detailed spec.** |
| `FLIGHT_DECK/PILOT_HANDBOOK.md` | How your work will be tested | Read once to understand expectations |
| `FLIGHT_DECK/FLIGHT_DEBRIEF.md` | How bugs will be reported back to you | Read once to understand fix format |
| `FLIGHT_DECK/PREFLIGHT_CHECK.md` | Testing depth settings — personas, tiers, thresholds | Read once to know current project testing level |
| `FLIGHT_DECK/TEST_FLIGHT_PROTOCOL.md` | How the test pilot tests screen by screen | Read once to understand the execution cycle |
| `FLIGHT_DECK/FLIGHT_LOG.md` | Cross-run metrics, Tier Gap tracking, UX phase testing | Read once to understand how UX quality is measured across tiers |
| `FLIGHT_DECK/INSIDER_TESTING_SKILL.md` | Insider testing methodology — Five Lenses, 8 phases, oracles, outcome verification | Read once. This is how Insider tier testing works. |
| `FLIGHT_DECK/INSIDER_GOALS.md` | Project-specific testing playbook (generated from Phase 0 Q&A) | **Test pilot reads before every test flight.** Created during Step 1.5. |
| This file (`AUTOPILOT.md`) | Your rules of engagement | You already loaded this |

---

## 🔓 Pre-Flight Permissions (Required — Do This First)

The Test Pilot Loop is an **autonomous build-test-fix cycle**. Claude Code must be able to edit files, run builds, and update FLIGHT_PLAN.md without stopping to ask the human for permission on every action. If permissions are not pre-configured, every tool call prompts the human — breaking the loop and defeating the purpose.

**In a real loop run (PhotoSortVision, 8 iterations, 19 bugs), Claude Code triggered 50+ permission prompts.** Each one stalled the loop until the human approved. That's not autonomous — it's a human approving an assembly line one screw at a time.

### What to Configure

Before saying "start test pilot loop," the human director must add these permissions to the project's `.claude/settings.local.json`:

```json
{
  "permissions": {
    "allow": [
      "Edit",
      "Write(FLIGHT_DECK/*)",
      "Write(.claude/agents/*)",
      "Bash(xcodebuild*)",
      "Bash(fswatch*)",
      "Bash(mkdir*)",
      "Bash(cp*)",
      "Bash(ls*)",
      "Bash(wc*)",
      "Bash(date*)",
      "Bash(which*)",
      "Bash(brew install fswatch)"
    ]
  }
}
```

Adjust for your build tool:

| Project Type | Build Permission |
|---|---|
| macOS/iOS (Xcode) | `Bash(xcodebuild*)` |
| Web (Node) | `Bash(npm*)`, `Bash(npx*)` |
| Python | `Bash(python*)`, `Bash(pip*)`, `Bash(pytest*)` |
| Rust | `Bash(cargo*)` |
| Go | `Bash(go*)` |
| General | `Bash(make*)`, `Bash(cmake*)` |

### Why This Matters

Without pre-approved permissions, the loop flow is:

```
Cowork writes feedback → fswatch fires → Claude Code wakes →
  → tries to edit file → ⏸️  BLOCKED: "Allow Edit?" → human approves →
  → tries to build → ⏸️  BLOCKED: "Allow xcodebuild?" → human approves →
  → tries to update FLIGHT_PLAN.md → ⏸️  BLOCKED: "Allow Write?" → human approves →
  → 3 approvals per fix × 5 bugs × 8 iterations = 120 interruptions
```

With pre-approved permissions:

```
Cowork writes feedback → fswatch fires → Claude Code wakes →
  → edits file → builds → updates FLIGHT_PLAN.md → done
  → total time: seconds, not minutes
```

### Cowork (Test Pilot) Permissions

Cowork uses computer use (`request_access`) to control apps on the user's Mac. Without pre-granting apps, every test flight triggers multiple permission dialogs — one per app, mid-test.

**At session start, Cowork must `request_access` for ALL apps it will need during the loop:**

| App | Why | Tier |
|-----|-----|------|
| **Your app** (e.g., PhotoSortVision) | Test the app through computer use | full |
| **Finder** | Verify folders, check file counts, open output | full |
| **Preview** | Inspect photos, verify transforms, check output | full |
| **Xcode** | Clean Build Folder, run builds (click-only) | click |
| **Terminal** | Monitor Claude Code progress (click-only) | click |

Also request: `clipboardRead`, `clipboardWrite`, `systemKeyCombos`.

**Do this in ONE `request_access` call at session start — not piecemeal during testing.** The human approves once, then the entire test flight runs uninterrupted.

Without pre-granted access:
```
Cowork starts test → opens app → ⏸️  "Allow PhotoSortVision?" → human approves →
  → checks Finder → ⏸️  "Allow Finder?" → human approves →
  → opens photo in Preview → ⏸️  "Allow Preview?" → human approves →
  → 3 prompts per test × 8 iterations = 24 interruptions
```

With pre-granted access:
```
Cowork starts test → opens app → clicks through → checks Finder → done
  → zero interruptions
```

### Security Note

These permissions are scoped to the project directory. Claude Code cannot modify files outside the project, run arbitrary network commands, or access system resources beyond what's listed. The human director retains kill-switch authority via `GLOBAL_STOP: FROZEN` in FLIGHT_PLAN.md.

Cowork's computer-use access is scoped to the granted apps only — it cannot see or interact with apps outside the allowlist. The human can revoke access at any time.

### Verification

After configuring permissions, test by running:
```
Claude Code:
  Edit a comment in any source file → should not prompt
  Run the build command → should not prompt
  Write to FLIGHT_DECK/FLIGHT_PLAN.md → should not prompt

Cowork:
  Open the app → should not prompt
  Open Finder → should not prompt
  Take a screenshot → should show all granted apps
```

If any of these prompt for approval, the permissions are not configured correctly. Fix before starting the loop.

---

## 🔁 Auto-Patrol Setup (Required)

After installing the framework, Claude Code **MUST** complete these setup steps. This is not optional — without them, the dual-patrol model doesn't work.

### Step 1: Read the Project Spec (MANDATORY)

Before writing any code or setting up patrol, Claude Code **MUST** read:
- `CLAUDE.md` — the project spec and state machine (every screen, button, toggle, state transition)
- `PRD.md` — the product requirements document (if it exists separately from CLAUDE.md)
- All Flight Deck files listed above (read once)

**Do NOT start building until you understand the full project.** The Test Pilot will test every function against the spec. If you don't know the spec, you'll build it wrong.

**On first setup:** If the Flight Deck files don't exist in your project yet, read them from the framework repo (the Test Pilot Loop source), then copy `AUTOPILOT.md` into your project's `FLIGHT_DECK/` and persona agent files into `.claude/agents/`. Update `CLAUDE.md` to reference `@FLIGHT_DECK/AUTOPILOT.md`.

### Step 1.5: Goal Discovery Q&A (MANDATORY — Insider Skill)

**After reading the spec but BEFORE setting up patrol**, Claude Code conducts a **6-question Q&A** with the human director. This generates `FLIGHT_DECK/INSIDER_GOALS.md` — the project-specific testing playbook that tells the test pilot what "correct" looks like, where to verify output, which datasets exist, and what the human is most worried about.

**Full protocol:** See `FLIGHT_DECK/INSIDER_TESTING_SKILL.md` → Phase 0.

**Quick summary:**
1. Claude Code generates Q1-Q5 from the spec (goal verification, failure investigation, test data, critical settings, out-of-scope features)
2. Q6 is always: "Why are you testing?"
3. Answers are compiled into `FLIGHT_DECK/INSIDER_GOALS.md`
4. The test pilot reads INSIDER_GOALS.md before every test flight

**Skip conditions:** Only skip if the human explicitly says to skip, or if the app is not yet built enough to produce meaningful output for verification.

### Step 2: Set Up Auto-Patrol (MANDATORY)

Claude Code runs on-demand per message, not as a persistent process. Without an explicit loop, it will never check `FLIGHT_PLAN.md` on its own.

### For Claude Code CLI:

Choose one patrol method. **fswatch is recommended** — it's event-driven, burns zero tokens while idle, and responds instantly when Cowork updates FLIGHT_PLAN.md.

#### Option 1: fswatch (RECOMMENDED — event-driven, zero waste)

```
run_in_background "fswatch -1 FLIGHT_DECK/FLIGHT_PLAN.md && echo 'FLIGHT_PLAN changed'"
```

Claude Code sleeps until FLIGHT_PLAN.md is modified, then wakes up, reads it, and acts. No polling, no wasted tokens. This is what a real CI/CD system would do.

When the watcher fires, Claude Code runs the patrol prompt:

```
"FLIGHT_PLAN.md changed — read the STATUS BLOCK and FEEDBACK QUEUE.
If STATUS is BUILD_REQUESTED or there is new feedback addressed to Claude Code:
  1. Read the feedback/task details
  2. Fix the bugs or build the feature
  3. Run tests, rebuild via xcodebuild or appropriate build tool
  4. Launch the app so Cowork can test it
  5. Write output to AGENT OUTPUT LOG
  6. Update STATUS → READY_FOR_TEST
If STATUS is ALL_BUGS_VERIFIED or PASS:
  1. Verify no stuck feedback, unanswered bugs, or pending tasks
  2. If all clear → write 'Patrol complete. Watcher stopped.' to AGENT OUTPUT LOG
  3. Stop watching — the cycle is done.
  4. If anything unresolved → write warning to FLIGHT_PLAN.md, keep watching.
If STATUS is READY_FOR_TEST → do nothing, waiting for Cowork.
If GLOBAL_STOP is FROZEN → stop all work, write frozen state to log.
Re-arm the watcher for the next change."
```

**Install fswatch if not present:** `brew install fswatch` (macOS) or `apt install fswatch` (Linux).

#### Option 2: /loop cron (FALLBACK — works everywhere, wastes tokens)

Use this if fswatch is not available or not practical for your setup:

```
/loop 10m "Patrol FLIGHT_DECK/FLIGHT_PLAN.md — read the STATUS BLOCK and FEEDBACK QUEUE.
If STATUS is BUILD_REQUESTED or there is new feedback addressed to Claude Code:
  1. Read the feedback/task details
  2. Fix the bugs or build the feature
  3. Run tests, rebuild via xcodebuild or appropriate build tool
  4. Launch the app so Cowork can test it
  5. Write output to AGENT OUTPUT LOG
  6. Update STATUS → READY_FOR_TEST
If STATUS is ALL_BUGS_VERIFIED or PASS:
  1. Verify no stuck feedback, unanswered bugs, or pending tasks
  2. If all clear → write 'Patrol complete. Loop stopped.' to AGENT OUTPUT LOG
  3. Cancel this /loop — the cycle is done. Do not keep polling.
  4. If anything unresolved → write warning to FLIGHT_PLAN.md, keep polling.
If STATUS is READY_FOR_TEST → do nothing, waiting for Cowork.
If GLOBAL_STOP is FROZEN → stop all work, write frozen state to log."
```

**Downside:** Checks every 10 minutes even when nothing changed. Burns tokens saying "standing by."

### Escalation Triggers (include in patrol prompt regardless of method):

Add these so Claude Code self-escalates:

- `GLOBAL_STOP: FROZEN` → alert human, halt all work
- Cowork stuck 30+ min with no update → write escalation to FLIGHT_PLAN.md
- Build fails 2x on the same fix → alert human before third attempt
- Bug fails retest 3x → alert human, likely architectural issue
- S1 Blocker found → alert human + pause immediately
- Confidence < 60% → alert human before marking ready for test

### For Cowork (Opus in desktop app):

Cowork patrol setup is **equally mandatory**. Cowork stays open as a persistent session with computer use, but it will NOT automatically patrol unless explicitly instructed. The human director **must** give Cowork the standing patrol instruction from `TEST_PILOT_LOOP.md` at session start. Without this, Cowork will sit idle and never pick up `READY_FOR_TEST` signals from Claude Code.

After receiving the patrol instruction, Cowork should write confirmation to FLIGHT_PLAN.md:

```
[timestamp] — Cowork Opus
Patrol configured: checking FLIGHT_PLAN.md every 10 minutes.
Standing by for READY_FOR_TEST signals.
```

### Verification — Both Agents:

**Neither agent can be assumed running until it confirms in writing.**

After setup, check the AGENT OUTPUT LOG in FLIGHT_PLAN.md for **both** entries:

```
[timestamp] — Claude Code
Auto-patrol configured: /loop 10m active.
Polling FLIGHT_PLAN.md for STATUS changes and feedback.

[timestamp] — Cowork Opus
Patrol configured: checking FLIGHT_PLAN.md every 10 minutes.
Standing by for READY_FOR_TEST signals.
```

If either entry is missing, that agent's patrol is not running. **Do not start building until both confirmations are present.** The loop requires two patrolling agents — one building, one testing. A single missing patrol breaks the entire feedback cycle.

### Patrol Lifecycle — Start, Stop, Restart

Patrols are not permanent. They follow the lifecycle of a test flight cycle.

**When to START patrol:**
- When `FLIGHT_PLAN.md` is created for a new cycle
- When the human says "start test pilot loop" or sets new tasks in the Task Queue
- Both agents must confirm in writing (see Verification above)

**When to AUTO-STOP patrol:**
- When STATUS reaches `ALL_BUGS_VERIFIED` or `PASS` — the cycle is complete
- **Cowork writes TEST FLIGHT COMPLETE summary first** (see `FLIGHT_DEBRIEF.md` → Test Flight Complete): all bugs fixed (table), all UX feedback collected (table with Cowork assessment), testing config used, timing. This is the handoff document for the human.
- Claude Code (fswatch): detect terminal status → write "Patrol complete. Watcher stopped." to AGENT OUTPUT LOG → stop the fswatch background process
- Claude Code (/loop fallback): detect terminal status → write "Patrol complete. Loop stopped." to AGENT OUTPUT LOG → cancel the `/loop` cron job
- Cowork: write summary → write "Patrol complete. Standing down." to AGENT OUTPUT LOG → stop checking
- **Before stopping, run escalation checks:** verify no stuck feedback, no unanswered bugs, no pending tasks. If anything is unresolved, do NOT stop — write a warning to FLIGHT_PLAN.md instead

**When to RESTART patrol:**
- Human explicitly says "test pilot loop" or "start next cycle"
- Human adds new tasks to the Task Queue and sets STATUS back to `BUILD_REQUESTED`
- Both agents must re-confirm patrol in writing before building resumes

**Why auto-stop matters:** Without it, both agents waste tokens polling an idle file indefinitely. A completed loop should clean up after itself.

---

## 🔄 Your Workflow — The Loop You Follow

```
1. CHECK STATUS
   → Read FLIGHT_PLAN.md — check GLOBAL STOP and FEEDBACK QUEUE
   → If GLOBAL STOP is 🛑 FROZEN → do nothing
   → If there's feedback for you → follow it before doing anything else

2. READ YOUR TASK
   → Find your assigned task in the TASK QUEUE
   → Note the risk level: ⚪ Trivial / 🟢 Low / 🟡 Medium / 🔴 High
   → Read the PRD section relevant to this task

3. WRITE YOUR PLAN
   → Before writing any code, write a brief plan in the AGENT OUTPUT LOG
   → Include: what you'll do, which files you'll touch, estimated complexity

4. BUILD
   → Execute your plan
   → Run tests. Fix failures. Iterate.
   → If using Ralph Wiggum Loop: the loop handles retry automatically

5. LAUNCH THE APP
   → After code compiles and tests pass, launch the app so Cowork can test it
   → macOS app: `open -a "YourApp"` or build + open
   → iOS app: build and install to Simulator
   → Web app: start dev server (`npm run dev`) if not already running
   → Make sure the app window is visible on screen, not minimized

6. REPORT OUTPUT + SIGNAL READY FOR TEST
   → Write results to AGENT OUTPUT LOG using the template below
   → Include your CONFIDENCE SCORE (0-100%)
   → Include: "App: Running at [window title / URL / Simulator]"
   → Include: "Ready for test pilot."
   → Cowork Opus will read this and begin testing

7. STOP OR CONTINUE (based on risk level)
   → ⚪ Trivial: set ⏳ Awaiting Review — Opus will fast-approve on next patrol
   → 🟢 Low + confidence ≥ 80% + next task also 🟢: auto-proceed
   → 🟢 Low + confidence < 80%: STOP — set ⏳ Awaiting Review
   → 🟡 Medium or 🔴 High: STOP — set ⏳ Awaiting Review
   → WAIT for feedback before starting next task (unless auto-proceeding)

8. RECEIVE FEEDBACK
   → Read FEEDBACK QUEUE for direction from Cowork Opus
   → If bugs found by Test Pilot: fix them specifically
   → After fixing: rebuild, relaunch the app, write "Fixed. App relaunched. Ready for retest."
   → If plan revision needed: revise and re-submit
   → Return to step 1
```

---

## 📝 Output Template — Use This Every Time

When you complete a task, append this to the AGENT OUTPUT LOG section of `FLIGHT_PLAN.md`:

```
[YYYY-MM-DD HH:MM] — Claude Code
Task ID: [T00X]
Risk Level: [⚪/🟢/🟡/🔴]

Plan:
  • Step 1: [what you planned to do]
  • Step 2: [what you planned to do]

Output:
  [what actually happened — result, changes made, or error]

Files Modified:
  • [path/to/modified-file]
  • [path/to/another-file]

Tests:
  [passed / failed — details if failed]

Confidence: [0-100%]
  [if < 80%, explain why — what are you unsure about?]

Status: [🟢 Auto-Proceeding to T00X / ⏳ Awaiting Review]

Self-Assessment:
  [anything the reviewer should know — edge cases not covered,
   design decisions you made, questions for the brain]
```

---

## 🎯 Confidence Score Guide

Your confidence score determines what happens next. Be honest.

| Score | Meaning | What Happens |
|-------|---------|-------------|
| 90-100% | Very confident. Clean implementation, tests pass, no concerns. | Auto-proceed if 🟢 Low |
| 80-89% | Confident with minor uncertainty. Works but has untested edges. | Auto-proceed if 🟢 Low, but flagged for batch review |
| 60-79% | Moderate confidence. Works for happy path, unsure about edge cases. | **STOP. Set ⏳ Awaiting Review regardless of risk level.** |
| Below 60% | Low confidence. May have fundamental issues. | **STOP. Flag for immediate Opus review.** |

**Rules:**
- Never inflate your confidence to avoid stopping
- If you had to make a design decision that wasn't in the PRD, confidence ≤ 80%
- If you're unsure whether your approach is architecturally correct, confidence ≤ 70%
- If tests pass but you feel something is "off," say so in Self-Assessment

---

## 🚫 Things You Must NOT Do

1. **Never start a task not in the Task Queue.** If you think a task is needed, flag it in your output. Don't build it.

2. **Never make architectural decisions.** If the task requires choosing between approaches, stop and ask. The brain (Cowork Opus) decides architecture.

3. **Never overwrite another agent's entries.** The AGENT OUTPUT LOG is append-only. Never delete or modify entries written by other agents.

4. **Never ignore the FEEDBACK QUEUE.** If there's feedback addressed to you, read and follow it before doing anything else.

5. **Never skip the plan.** Even for simple tasks, write what you intend to do before doing it. This is how the system audits your work.

6. **Never continue after a 🟡/🔴 task without feedback.** Stop. Wait. The Test Pilot needs to use the app before you build on top of it.

---

## 🔁 When You Receive Bug Reports From the Test Pilot

The Test Pilot will test your work by actually using the app. When bugs are found, they arrive in this format:

```
BUG-T004-001 [S3 🟡 Major]: "Import" requires double-tap
  Steps: Open app → Tap "Import" → nothing happens → tap again → works
  Expected: Import starts on first tap
  Actual: Requires double-tap
  Suggested fix: event handler conflict with component init
```

**How to handle bug reports:**
- Fix bugs in the order they're listed (highest severity first)
- Don't rewrite unrelated code while fixing bugs
- After fixing, write a new output entry with what you changed
- Set status → ⏳ Awaiting Review for retest
- The Test Pilot will retest only the failed dimensions

---

## ⚡ Ralph Wiggum Loop Integration

If this project uses Ralph Wiggum Loop for persistent iteration:

- Ralph handles retry logic — you don't need to worry about re-running on failure
- Your `<promise>COMPLETE</promise>` tag only exits Ralph when code tests pass
- But that's NOT the end — the Test Pilot gate still needs to pass
- If Test Pilot fails, Opus updates the prompt with UX findings and Ralph loops again
- You'll see the updated prompt with specific bugs and UX feedback to address

**Your job in a Ralph loop:**
1. Read the prompt (may include Test Pilot findings from previous iteration)
2. Build or fix what's described
3. Make tests pass
4. Output `<promise>COMPLETE</promise>` when code is correct
5. The system handles the rest (Test Pilot validation, feedback loop)

---

## 🛑 Kill Switch

If you see `GLOBAL STOP: 🛑 FROZEN` in the STATUS BLOCK:
- **Stop immediately.** Do not start, continue, or finish any work.
- Write your current state to AGENT OUTPUT LOG with status `🛑 Frozen`
- Wait for `GLOBAL STOP: ACTIVE` before resuming

---

## 🤝 Who's Who in This System

| Agent | Role | Model | How They Affect You |
|-------|------|-------|-------------------|
| **Cowork Opus** | The brain. Patrols every 10 min. Test-pilots the app. Makes all decisions. | Opus | Writes your tasks, reviews your output, tests the app, sends you feedback |
| **Test Pilot Personas** | Optional: UX testers with specific behavioral rules. | Sonnet | If deployed, test your work as different user types. Report UX bugs back to you. |
| **Second Builder** | Another builder agent (if active). | Sonnet | Works on different tasks in parallel. Don't touch their files. |
| **Human Director** | Your human supervisor. | — | Has final authority. Opus escalates to the human director for critical decisions. |

---

## 💡 Writing Code for the Test Pilot

Your code will be tested by an AI that operates the app through computer use, evaluating it at different skill levels. Write defensively:

**Labels:** Make every button, field, and section self-explanatory. If you need a comment to explain what a button does, the label is wrong.

**Feedback:** After every user action, the UI should confirm what happened. Tap save → "Saved." Tap delete → "Are you sure?" Then "Deleted." Silence after an action is a UX bug.

**Error messages:** Write for the confused user, not the developer. "Network request failed (HTTP 403)" is bad. "Couldn't connect. Check your internet and try again." is good.

**Touch targets:** Minimum 44x44pt on iOS, 48x48dp on Android. The struggling user has big fingers and shaky hands.

**Empty states:** What does the screen look like with no data? "No items yet. Tap + to start." is good. A blank white screen is a UX bug.

**Undo:** If an action is destructive (delete, overwrite, discard), provide undo or confirmation. The struggling user will accidentally tap things.

**Consistency:** If one screen says "Cancel" and another says "Back" and a third says "Close," that's three different words for the same action. Pick one.

---

## 📋 Project-Specific Context

> **Add your project details below this line.**
> The section above is universal. The section below is yours.

### Project: [YOUR PROJECT NAME]
### PRD: [path/to/PRD.md]
### Platform: [iOS / macOS / Web / etc.]
### Tech Stack: [Swift / React / Python / etc.]
### Key Conventions:
- [your coding style rules]
- [your naming conventions]
- [your file organization rules]
- [anything Claude Code should always follow in THIS project]

---

*Autopilot — Test Pilot Loop — Flight Deck*
*Universal template. Works with any project. Teaches Claude Code the discipline.*
