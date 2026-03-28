# 🧪 Test Pilot Loop
### One Brain, Three Knowledge Levels, Continuous Loop
### Cowork Opus stays live. Tests, feeds back, retests — until it passes.

---

## What This Is

Cowork Opus stays open as the **brain of your project**. After your code agent builds a feature, Opus test-drives the app three times — each time constrained to a different knowledge level. The gap between those experiences tells you how intuitive your app really is.

Then Opus sends structured feedback to the code agent. The code agent fixes. Opus retests. **The loop stays in one continuous session until the feature passes.**

**The app isn't done when code passes. It's done when someone who knows nothing can use it.**

---

## Why Three Knowledge Levels

The developer knows every button. A new user knows nothing. The gap between them is your UX problem.

| Run | Knowledge Level | What They Get | Simulates |
|-----|----------------|---------------|-----------|
| **Run 1: Cold** | Knows NOTHING | App name only. No docs, no PRD, no hints. | User who just downloaded the app |
| **Run 2: Guided** | Has the manual | User manual or README only. No PRD. | User who reads the docs first |
| **Run 3: Insider** | Has the PRD | Full product requirements. Knows everything. | Product manager doing acceptance testing |

Same Opus instance, but each run is **prompted with only that tier's allowed knowledge**. The cold run is the hardest — and the most valuable.

**The Tier Gap:** If the Cold user takes 12 steps and the Insider takes 4, that 3x gap IS your UX debt. When you close that gap, your app is intuitive.

---

## The Loop

```
┌─────────────────────────────────────────────────────────┐
│              🟣 COWORK OPUS (Brain — stays open)         │
│                                                          │
│  CODE AGENT builds feature                               │
│  Writes to FLIGHT_PLAN.md: "Ready for test"            │
│           ↓                                              │
│  OPUS reads the file, sees "Ready for test"              │
│           ↓                                              │
│  Run 1: COLD — Opus tests knowing NOTHING                │
│    → Records: steps, confusion, abandonment points       │
│    → Reset app                                           │
│           ↓                                              │
│  Run 2: GUIDED — Opus tests with user manual only        │
│    → Records: doc accuracy, terminology gaps             │
│    → Reset app                                           │
│           ↓                                              │
│  Run 3: INSIDER — Opus tests with full PRD               │
│    → Records: spec compliance, feature completeness      │
│           ↓                                              │
│  OPUS writes FEEDBACK to FLIGHT_PLAN.md:               │
│    • UX issues from all three runs                       │
│    • Tier Gap Ratio (Cold steps / Insider steps)         │
│    • Specific fixes needed                               │
│    • Verdict: PASS / NEEDS WORK / FAIL                   │
│           ↓                                              │
│  CODE AGENT reads feedback → fixes → writes:             │
│    "Fixed. Ready for retest."                            │
│           ↓                                              │
│  OPUS retests (same three runs)                          │
│           ↓                                              │
│  LOOP until Opus says PASS for all three levels          │
│           ↓                                              │
│  ✅ SHIP IT                                              │
└─────────────────────────────────────────────────────────┘
```

---

## The Dual-Patrol Model

Cowork and the code agent are **two independent processes** that never interact directly. They communicate exclusively through a shared file: `FLIGHT_DECK/FLIGHT_PLAN.md`.

**Cowork Opus** is the brain and test pilot. It patrols `FLIGHT_PLAN.md` every 10 minutes, looking for "Ready for test." When it finds one, it switches to test mode — finds the app on screen, runs the three knowledge-level tests through computer use, writes structured feedback back to the file, and returns to patrol.

**Claude Code** (or any coding agent) is the builder. It builds features, runs tests, launches the app, and writes "Ready for test" to the shared file. Then it polls the same file on its own 10-minute cycle. When it finds feedback and a `NEXT_ACTION`, it reads the instructions, fixes the issues, rebuilds, relaunches, and writes "Ready for retest."

Neither agent commands the other. Cowork cannot type into Terminal (platform safety boundary), and doesn't need to — the file is the only coordination channel. If an agent stops responding, the other writes an escalation to the file and the human director intervenes.

This architecture is portable: any coding agent that can read and write a markdown file (Claude Code, Codex CLI, Cursor, etc.) can serve as the builder side.

### The Communication Channel

```
CODE AGENT writes:
──────────────────
[2026-03-27 10:30] — Claude Code
Task: Photo import feature
Status: ✅ Built. Code compiles. Tests pass.
Ready for test pilot.

OPUS writes back:
──────────────────
[2026-03-27 10:35] — Cowork Opus Test Pilot

RUN 1 — COLD USER (knows nothing):
  Completed: NO — gave up at step 8
  Steps taken: 8
  Confusion: Couldn't find the import button.
  The icon looks like a camera but feature is called "Import."

RUN 2 — GUIDED USER (has manual):
  Completed: YES
  Steps taken: 6
  Doc issue: Manual says "tap Import on main screen"
  but the button is on the second tab.

RUN 3 — INSIDER (has PRD):
  Completed: YES
  Steps taken: 3
  Spec compliance: 90% — missing the "drag-and-drop" 
  import mentioned in PRD section 4.2.

TIER GAP RATIO: Cold 8 steps / Insider 3 steps = 2.7x
TARGET: ≤ 1.5x

FEEDBACK FOR CODE AGENT:
  1. Rename camera icon to "Import" with text label
  2. Move import button to main screen (not second tab)
  3. Add drag-and-drop import per PRD section 4.2
  4. Add onboarding tooltip pointing to import on first launch

VERDICT: ❌ NEEDS WORK — fix items above, then signal ready for retest.

CODE AGENT writes back:
──────────────────────
[2026-03-27 11:00] — Claude Code
  Fixed items 1-4. Import button moved to main screen
  with text label. Drag-and-drop added. Tooltip added.
  Status: ✅ Fixed. Ready for retest.

OPUS retests...
```

---

## How to Run

### Session Start

Launch both apps at the same time on the same Mac:

1. **Open Cowork with Opus** — give it the standing patrol instruction (below)
2. **Open Claude Code** (or any AI coding agent) — start building
3. **Create `FLIGHT_DECK/FLIGHT_PLAN.md`** in your project — this is the shared communication file

### The Opus Patrol Instruction

Give Cowork Opus this prompt at the start of your session:

```
You are the test pilot and patrol monitor for this project.

YOUR TWO MODES:

PATROL MODE (default):
  Every 10 minutes, read FLIGHT_PLAN.md in the project folder.
  Look for "Ready for test" or "Ready for retest."
  If nothing new → go back to waiting. Keep it lightweight.

TEST MODE (triggered by "Ready for test"):
  When you see "Ready for test" or "Ready for retest":
  1. Read the details — what was built, where the app is running
  2. Find the app on screen (window title, URL, or Simulator)
  3. Run three knowledge-level tests (Cold, Guided, Insider)
  4. Write your structured feedback to FLIGHT_PLAN.md
  5. Update FLIGHT_PLAN.md with specific instructions in NEXT_ACTION_FOR_CLAUDE_CODE. Claude Code picks this up on its next patrol cycle (every 10 minutes).
  6. Return to PATROL MODE

Keep cycling between patrol and test until I tell you to stop.
This session may run overnight — pace yourself.
```

**Why 10 minutes?** Designed for overnight operation. Nobody is waiting. For daytime use, tell Opus "check every 5 minutes" or simply say "check the file now."

### How the Code Agent Launches the App

After building, Claude Code launches the app so Opus can find it on screen:

| Project Type | Claude Code Runs |
|---|---|
| **macOS app** | `open -a "YourApp"` |
| **iOS app** | Build + install to Simulator via `xcodebuild` |
| **Web app** | `npm run dev` (keeps server running) |
| **Website** | `open https://your-url.com` |
| **Electron app** | `npm start` |

Then writes to the shared file:

```
[2026-03-27 10:30] — Claude Code
Task: Photo import feature
Status: ✅ Built. Tests pass.
App: Running — window titled "PhotoSortVision" on desktop
Ready for test pilot.
```

Opus picks this up on the next patrol check.

### How Cowork Launches the App for Testing

For Xcode projects, Cowork Opus can build and launch the app directly by clicking menu items (click-tier access is sufficient for Xcode menus). The typical flow:

1. Claude Code builds the feature and writes "Ready for test" to FLIGHT_PLAN.md
2. Opus sees this and finds the Xcode window on screen
3. **For macOS apps:** Opus clicks Product → Build, then Product → Run to launch the app
4. Alternatively, Claude Code can build via `xcodebuild` command line and place the `.app` in a known location (e.g., `~/Downloads/`). Opus then launches it with `open -a "AppName"` or by clicking the `.app` in Finder
5. Once the app is running, Opus can test it directly

This avoids any Terminal typing. The build happens either through Xcode menu clicks or through Claude Code's earlier command line invocation.

### Dual-Patrol in Detail

Both agents patrol `FLIGHT_PLAN.md` independently every 10 minutes. Neither agent needs to interact with the other directly — they communicate purely through the shared file using STATUS codes and NEXT_ACTION instructions.

```
Session Start: User opens Cowork (Opus) + Claude Code
                       ↓
┌──────────────────────────────────────────────────────┐
│                                                       │
│  Claude Code              Cowork Opus                 │
│  ────────────             ───────────                 │
│  Builds feature           PATROL MODE                 │
│  Runs tests               (checks FLIGHT_PLAN.md      │
│  Launches app             every 10 min)               │
│  Writes "Ready" ──────→ FLIGHT_PLAN.md ←── polls     │
│                       (status: READY_FOR_TEST)        │
│                                                       │
│                       Switches to TEST MODE           │
│                       Finds app on screen             │
│                       Tests 3 knowledge levels        │
│                       Writes feedback +               │
│                       NEXT_ACTION to file             │
│                                                       │
│                       ↓ (both agents poll)            │
│  Polls FLIGHT_PLAN.md ◄─ Claude Code picks up        │
│  Reads NEXT_ACTION        NEXT_ACTION on next          │
│  (every 10 min)           patrol (every 10 min)        │
│                                                       │
│  Fixes issues                                         │
│  Rebuilds + relaunches                                │
│  Writes "Ready for retest" ──→ Opus polls             │
│                                  Sees status change    │
│                                  TEST MODE again       │
│                                                       │
│  LOOP until Opus writes PASS                          │
└──────────────────────────────────────────────────────┘
```

**The Dual-Patrol architecture:** Both agents independently patrol the shared file. Cowork cannot type into Terminal (click-tier safety boundary). The Dual-Patrol architecture routes around this entirely — neither agent needs to interact with the other directly. Communication is purely through STATUS codes and plain-language instructions in `NEXT_ACTION_FOR_CLAUDE_CODE`.

### The Cycle

**Code agent's job:**
1. Build the feature
2. **Launch the app** (terminal command)
3. Write to `FLIGHT_PLAN.md`: "App running at [location]. Ready for test."
4. Wait for instructions by polling `FLIGHT_PLAN.md` (reads on own patrol cycle every 10 minutes)
5. When Opus writes feedback and NEXT_ACTION → read `FLIGHT_PLAN.md` for details
6. Fix issues, rebuild, relaunch app
7. Write: "Fixed. App relaunched. Ready for retest."
8. Repeat until Opus says PASS

**Cowork Opus's job:**
1. **Patrol** `FLIGHT_PLAN.md` every 10 minutes
2. When "Ready for test" appears → switch to test mode
3. **Find the app on screen** (window title, URL, or Simulator)
4. Run three knowledge-level tests
5. Write structured feedback to `FLIGHT_PLAN.md`
6. **Write instructions to FLIGHT_PLAN.md → Claude Code reads on next patrol:**
   Update `NEXT_ACTION_FOR_CLAUDE_CODE` with plain-language instructions like: *"Read FLIGHT_PLAN.md for test pilot feedback. Fix the issues 1-3, rebuild, relaunch, and update the file when ready for retest."*
7. Return to patrol mode
8. When "Ready for retest" appears → test again
9. Approve when all three levels pass

### The Three Test Runs

**Run 1: Cold User (knows nothing)**

Opus prompt to itself:
```
You have no prior knowledge of this project. You are constrained
to knowing only the app name. You don't know what it does, how
it works, or what any button means.

THE APP: [how to find it on screen — e.g., "The app window titled 
PhotoSortVision on the desktop" or "Open Safari and go to 
http://localhost:3000" or "The iOS Simulator on the right side 
of the screen"]

Your task: [what a user would try to do]

Use the app. Think out loud. Count every step. Note every 
moment of confusion. If you can't figure it out after 8 steps
of confusion, report abandonment.
```

**Run 2: Guided User (has manual only)**

Opus prompt to itself:
```
You have read the user manual (below). You have NOT seen the
PRD or any developer notes. Follow the manual's instructions
and note every place where the manual is wrong, incomplete,
or uses different words than the app.

[paste user manual / README]

Your task: [same task as Run 1]
```

**Run 3: Insider (has full PRD)**

Opus prompt to itself:
```
You have the full Product Requirements Document (below). You know
every intended feature and acceptance criterion. Test whether the
app delivers what was specified.

[paste PRD.md — product requirements and acceptance criteria]

Your task: [same task as Run 1]

For each requirement: does the app meet it? YES / PARTIALLY / NO.
Note any behaviors that differ from the spec.
```

---

## The Report Template

Opus fills this out after all three runs:

```
TEST PILOT REPORT — Three Knowledge Levels
═══════════════════════════════════════════
Feature: [what was built]
Task: [what was tested]

┌──────────────────────────────────────────────────────────┐
│                RUN 1         RUN 2         RUN 3         │
│                COLD          GUIDED        INSIDER       │
│                (nothing)     (manual)      (PRD)         │
├──────────────────────────────────────────────────────────┤
│ Completed:     [YES/NO]      [YES/NO]      [YES/NO]     │
│ Steps taken:   [count]       [count]       [count]       │
│ Wrong turns:   [count]       [count]       [count]       │
│ Confusion pts: [count]       [count]       [count]       │
│ Confidence:    [1-7]         [1-7]         [1-7]         │
└──────────────────────────────────────────────────────────┘

TIER GAP RATIO: [Cold steps] / [Insider steps] = [X]x
  ≤ 1.5x = ⭐ Excellent (intuitive, no manual needed)
  ≤ 2.0x = 🟢 Good (most users figure it out)
  ≤ 3.0x = 🟡 Acceptable (usable with documentation)
  > 3.0x = 🔴 Poor (too dependent on prior knowledge)

────────────────────────────────────────
COLD USER FINDINGS (most important)
────────────────────────────────────────
What confused me:
  • [moment — where, what, why]
  • [moment — where, what, why]

Where I would give up:
  • [exact point and reason]

What would have helped:
  • [specific suggestion]

────────────────────────────────────────
GUIDED USER FINDINGS (documentation quality)
────────────────────────────────────────
Manual accuracy:
  • [instruction that was wrong or outdated]
  • [feature not covered by manual]
  • [terminology mismatch: manual says X, app says Y]

────────────────────────────────────────
INSIDER FINDINGS (spec compliance)
────────────────────────────────────────
PRD compliance: [X]% of requirements met
  • [requirement not met — what's missing]
  • [behavior that differs from spec]
  • [PRD should be updated because — real usage insight]

────────────────────────────────────────
INTERFACE EVALUATION (across all runs)
────────────────────────────────────────
Layout clear:         YES / MOSTLY / NO
Labels make sense:    YES / MOSTLY / NO
Feedback after taps:  YES / SOMETIMES / NEVER
Know what to do next: YES / SOMETIMES / NO
Would use this app:   YES / MAYBE / NO

────────────────────────────────────────
BUGS FOUND
────────────────────────────────────────
  • [what happened vs what should have happened]

────────────────────────────────────────
FEEDBACK FOR CODE AGENT
────────────────────────────────────────
Fix these (priority order):
  1. [most important fix — what and why]
  2. [next fix]
  3. [next fix]

VERDICT: PASS ✅ / NEEDS WORK ⚠️ / FAIL ❌
```

---

## The Retest Trigger

After the code agent fixes issues, it writes to the shared file:

```
[timestamp] — Claude Code
Fixed items 1-3 from test pilot feedback.
Changes made:
  • Renamed button from "Execute" to "Start Import"
  • Added success message after import completes
  • Moved import button to main tab
Status: ✅ Fixed. Ready for retest.
```

Cowork Opus reads this, retests all three levels, and writes a new report. The loop continues until:

- **Cold user completes the task** (the most important gate)
- **Tier Gap Ratio ≤ 2.0x** (acceptable intuitiveness)
- **No bugs found**
- **PRD compliance ≥ 90%**

When all conditions are met: **PASS. Ship it.**

---

## When to Simplify

You don't always need all three levels:

| Situation | Use |
|-----------|-----|
| Quick check after small fix | **Cold only** — can the user still figure it out? |
| Documentation update | **Guided only** — does the manual still match? |
| Spec compliance check | **Insider only** — does it meet requirements? |
| Pre-release milestone | **All three** — full validation |

---

## Why This Works

**One session. No switching.** Cowork Opus stays live the whole time. It knows the project context, remembers previous test results, and tracks improvement across retests.

**The code agent drives the loop.** It signals "ready for test" and "ready for retest." Opus responds. The shared file is the communication channel. No manual coordination needed.

**The Tier Gap Ratio is a real metric.** If Cold takes 12 steps and Insider takes 4, that's 3.0x — your app depends heavily on prior knowledge. When it drops to 1.5x, your app is genuinely intuitive. You can track this across builds.

**You're measuring what matters.** Not "does the code work?" but "can a person who knows nothing about this app figure it out?"

---

## Scaling Up

When you're ready for more:

- **Quick Flight** → Want a faster check with zero setup? Use the 3-Model Quick Test where Opus, Sonnet, and Haiku each test your app independently. See `QUICK_FLIGHT.md`.
- **Test Flight Protocol** → Want the detailed screen-by-screen execution protocol? How to inspect, inventory, predict, act, and evaluate on every screen — including CLI-Anything hybrid testing and debug overlays. See `FLIGHT_DECK/TEST_FLIGHT_PROTOCOL.md`.
- **Add personas** → Instead of one Opus testing as "cold user," deploy persona subagents (Alex the power user, Pat the struggling user, Sam the accessibility user) for deeper behavioral testing. See `PERSONAS/`.
- **Add archetypes** → Layer Creator/Admin/Consumer behaviors on top of knowledge levels based on which feature is being tested.
- **Add orchestration** → Risk tiers, kill switch, Opus patrol, Ralph Wiggum build loop, confidence scores. See `FLIGHT_DECK/`.

All layers are compatible. Start here. Add rigor as your project grows.

---

*Test Pilot Loop v0.1.0*
*One brain. Three knowledge levels. Continuous loop.*
*Part of the Test Pilot Loop Framework by Huan Su.*
