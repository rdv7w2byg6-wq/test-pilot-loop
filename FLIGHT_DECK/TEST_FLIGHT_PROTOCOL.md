# ✈️ Test Flight Protocol — How the Test Pilot Tests Your App
### The step-by-step execution protocol for screen-by-screen testing through computer use.

---

## What This Is

Every other document tells you **what** to test and **who** to test as. This document tells you **how.**

This is the execution protocol the Test Pilot follows when operating a real app through computer use — how to inspect each screen, analyze every element, form predictions, take actions, and evaluate outcomes.

---

## The Core Cycle

Every screen gets this treatment. No exceptions.

```
📸 SEE        — Screenshot the full screen
📋 INVENTORY  — List every element
🧠 PREDICT    — What does each one do?
👆 ACT        — Choose one action, note why
✅ EVALUATE   — Match prediction?
🔄 REPEAT     — New screen, start over
```

---

## Three Verification Layers

| Layer | Method | Speed | Coverage | Best For |
|-------|--------|-------|----------|----------|
| **Layer 1: CLI** | Terminal commands, API calls | Seconds | Data state, errors, health | Phase 0 ground checks, Mode C verification |
| **Layer 2: Accessibility API** | macOS Accessibility Inspector, Playwright `accessibility.snapshot()`, iOS AX | Seconds | Labels, focus order, element roles, navigation structure | Label completeness, keyboard nav, screen reader compatibility |
| **Layer 3: Computer Use** | Screenshots, clicks, typing | Minutes | Full visual UX, layout, flows | Phases 1–3 flight testing |

Use the cheapest layer that answers the question. Layer 2 covers ~60% of usability checks (labels, focus order, element roles) at Layer 1 speed.

### Layer 2: Accessibility API Quick Reference

| Platform | Tool | Command |
|----------|------|---------|
| **macOS** | Accessibility Inspector | Xcode → Open Developer Tool → Accessibility Inspector |
| **Web** | Playwright | `page.accessibility.snapshot()` — returns full element tree with roles and names |
| **iOS** | XCUITest AX | `XCUIApplication().accessibilityElements()` |
| **Web (standalone)** | pa11y | `npx pa11y http://localhost:3000 --reporter json` |

**What Layer 2 catches without screenshots:**
- Buttons with no accessible label → usability bug
- Missing focus order → keyboard users can't navigate
- Elements with wrong roles (div acting as button) → assistive tech breaks
- Duplicate IDs → automation selectors fail

---

## Hybrid Testing: CLI First, Accessibility Second, Computer Use Third

Computer use is slow. Many checks are faster through CLI or accessibility APIs.

```
PHASE 0: GROUND CHECK (CLI + Accessibility)   ← Seconds
PHASE 1–3: FLIGHT (Computer Use)              ← Minutes
```

Do CLI and accessibility checks first. Catch the easy stuff fast. Spend the expensive computer-use time on what only visual inspection can evaluate.

### CLI-Anything Integration (Optional)

[CLI-Anything](https://github.com/HKUDS/CLI-Anything) auto-generates a CLI wrapper for any software — every button, action, and feature becomes a terminal command.

```bash
# Generate and install (one time per app)
/cli-anything ./your-app
cd your-app/agent-harness && pip install -e .

# Now every feature is a CLI command
cli-anything-yourapp --json status
cli-anything-yourapp --json list-features
```

The generated CLI is used in three places:

1. **Phase 0** — Fast ground checks (is the app running, any errors, data state)
2. **Mode C during testing** — Verify data actually changed after each visual action
3. **Phase 3 cleanup** — Discover all commands via `--help`, find features that exist in code but aren't discoverable in the UI

CLI-Anything is optional. All phases work with hand-written CLI commands.

---

## Pre-Flight: Visible Window Check

Before any testing begins, confirm the target app is visible and ready for computer use interaction.

```
VISIBLE WINDOW CHECK
════════════════════
□ App is running (not just built — actually launched)
□ App window is visible on screen (not minimized, not behind other windows)
□ App window is large enough to interact with (not a sliver)
□ For iOS: Simulator is in foreground with the app active
□ For web: Browser tab is active and page has loaded
□ For CLI apps: Terminal window is visible with the app's output showing

If any check fails → stop. Ask the human director to bring the app
window to the foreground, or ask Claude Code to relaunch it.
```

**Why this matters:** The Test Pilot operates through computer use — it can only see and interact with what's on screen. A minimized window, a background tab, or an app that built but never launched will produce a failed test with zero useful signal.

---

## Phase 0: Ground Check (CLI + Accessibility API)

Run before launching visual inspection. Each takes seconds.

**With CLI-Anything:**
```bash
cli-anything-yourapp --json status
cli-anything-yourapp --json health-check
cli-anything-yourapp --json list-features
# If any return errors → BLOCKER. Fix before visual testing.
```

**Without CLI-Anything, check manually:**

| Check | What to verify |
|-------|---------------|
| **App running?** | `curl -s -o /dev/null -w "%{http_code}" http://localhost:3000` or `pgrep -x "YourApp"` |
| **Errors in logs?** | `tail -50 app.log \| grep -i "error\|fatal\|crash"` |
| **Data state?** | `sqlite3 app.db "SELECT COUNT(*) FROM [table];"` or check API |
| **Accessibility (violations)?** | `npx pa11y http://localhost:3000` |
| **Accessibility (structure)?** | Layer 2 check — see Three Verification Layers above |
| **Performance?** | `curl -s -o /dev/null -w "Total: %{time_total}s" http://localhost:3000` |

```
GROUND CHECK SUMMARY
════════════════════
App running:     ✅ / ❌
Errors in log:   [count]
Data state:      OK / ISSUES
Accessibility:   [violation count]
Performance:     [load time]s

BLOCKERS: [must fix before visual testing]
WARNINGS: [note but continue]
Ready for visual inspection: YES / NO
```

If blockers found: stop. Code agent fixes. Rerun ground check.

---

## Phase 1: Preflight Inspection (First Screen)

Before touching anything.

### 1.1 Screenshot
Take a full screenshot on first launch.

### 1.2 Element Inventory
List **every visible element** — be exhaustive.

```
ELEMENT INVENTORY — Screen #1
══════════════════════════════
NAVIGATION:
  [1] Top-left: [icon/button]
  [2] Top-center: [title text]
  [3] Top-right: [icon/button]

INTERACTIVE ELEMENTS:
  [4] Button: "[label]" — [color, size, position]
  [5] Input field: "[placeholder]"
  [6] Icon button: [describe] — [labeled / unlabeled]

TOTAL INTERACTIVE: [count]
LABELED: [count]  UNLABELED: [count]
```

### 1.3 Predictions (Tier-Dependent)

For each interactive element, predict what it does:

| Tier | Prediction basis | Finding type |
|------|-----------------|--------------|
| **Cold** | Icons, labels, layout patterns | "This is confusing" |
| **Guided** | Does the manual describe this element? | "Manual is wrong/incomplete" |
| **Insider** | Does the PRD require this element? | "Spec not met / scope creep" |

### 1.4 First Impression

```
FIRST IMPRESSION
════════════════
What I think this app does: [guess]
Primary action the screen wants me to take: [guess]
Confidence: OBVIOUS / THINK SO / NO IDEA
Visual clarity: CLEAN / BUSY / CLUTTERED / EMPTY
```

### 1.5 Early Abort Gate (Cold Tier Only)

If the Cold User cannot identify the primary action within 5 attempts, the app has a fundamental discoverability problem. Stop the test early — further testing won't produce useful signal.

```
EARLY ABORT CHECK
═════════════════
After 5 actions:
  ✅ Found primary action → continue to Phase 2
  ❌ Still lost → ABORT — file P0 discoverability finding

Checkpoints:
  • 30 seconds — Can the user identify what the app does?
  • 2 minutes  — Can the user start the primary task?
  • 5 minutes  — Can the user complete the primary task?

If any checkpoint answer is NO for Cold tier → flag as finding.
If 30-second checkpoint is NO → strong candidate for early abort.
```

Early abort applies only to Cold tier. Guided and Insider tiers have external context and should complete the full test.

---

## Phase 2: Flight (Active Testing)

One action at a time. For each action:

### The Action Cycle

```
1. Choose element → note WHY (based on knowledge tier)
2. Predict what will happen
3. Screenshot BEFORE
4. Execute the action
5. Screenshot AFTER
6. Evaluate (Modes A, B, C)
7. If new screen → repeat from Phase 1
```

### Three Evaluation Modes

#### Mode A: Pure User (Default — Always Run)

Evaluate exactly what a user would see. Pixels only.

```
RESULT #[n]
═══════════
Visual changes: [what appeared / disappeared / changed]
Matched prediction: YES / NO / PARTIALLY
Feedback received: YES / NO
  If NO: ⚠️ "No feedback after tapping [element]"
Response time: INSTANT / BRIEF / SLOW / NONE
  If SLOW or NONE: ⚠️ "App felt unresponsive"
```

#### Mode B: Debug Overlay (Optional — Deeper Testing)

The code agent adds a visible debug panel showing internal state:

```
┌─────────────────────────────────────┐
│ 🔧 DEBUG OVERLAY (remove for prod)  │
│ Last action: [element tapped]       │
│ Event fired: [event name]           │
│ State: [old → new]                  │
│ Network: [URL → status code]        │
│ Error: [none / message]             │
│ Data saved: [YES/NO — what]         │
└─────────────────────────────────────┘
```

After each action, compare visual result to debug overlay:
- Visual says success + debug says success → ✅
- Visual silent + debug says success → ⚠️ **Hidden success** (feedback bug)
- Visual says error + debug says success → ⚠️ **False error** (UX bug)

#### Mode C: CLI Verification (After Data-Changing Actions)

```
CLI VERIFY after Action #[n]
════════════════════════════
Expected: [what should have changed]
$ cli-anything-yourapp --json [check command]
Result: [matches screen? YES / NO]
  If NO: ⚠️ DATA BUG — "Screen says [X] but data shows [Y]"
```

### What Each Mode Catches

| Mode | Speed | Catches |
|------|-------|---------|
| **A: Pure User** | Slow (screenshot) | Confusion, missing feedback, unclear labels |
| **B: Debug Overlay** | Slow (screenshot) | Silent successes, false errors, state mismatches |
| **C: CLI Verify** | Fast (terminal) | Data not saved, files not created, API failures |

Mode A is primary. B and C are supplementary. Never skip A.

---

## Phase 3: Cleanup Pass

After completing (or failing) the main task, test what you didn't touch.

**Untested elements:** Review your inventories. Which elements did you never tap? Tap each one, record what happens.

**Hidden interactions:** Try on every screen:
- Long-press on main elements
- Swipe left/right on list items
- Pull-to-refresh
- Right-click (desktop) / keyboard shortcuts

Record any hidden features and whether they were discoverable.

---

## Phase 4: Debrief

Compile the full report:

```
TEST FLIGHT REPORT
══════════════════
App: [name]
Tester: [Cold / Guided / Insider]
Task: [what was attempted]
Completed: YES / NO / PARTIALLY

GROUND CHECK: App ✅/❌ | Errors: [n] | A11y: [n] | Perf: [t]s

SCREENS VISITED: [n]
TOTAL ACTIONS: [n]
WRONG TURNS: [n]
CLI VERIFICATIONS: [passed]/[total]

ELEMENT ANALYSIS:
  Total interactive: [n]
  Clear labels: [n] ([%])
  No feedback when tapped: [list]
  Did something unexpected: [list]

PREDICTION ACCURACY: [correct]/[total] ([%])
  Wrong predictions = your UX problems

NAVIGATION: Can always go back: Y/N | Always know location: Y/N

TOP FINDINGS (priority order):
  1. [what, where, impact]
  2. ...

VERDICT: PASS ✅ / NEEDS WORK ⚠️ / FAIL ❌
```

---

## How Knowledge Tier Changes the Cycle

The cycle is identical for all tiers. Only the PREDICT and EVALUATE steps change:

| Step | Cold | Guided | Insider |
|------|------|--------|---------|
| **PREDICT** | Guess from labels and layout | Check manual for this element | Check PRD for this requirement |
| **EVALUATE** | "Did it do what I guessed?" | "Did it match the manual?" | "Did it meet the spec?" |

---

## Timing

| Phase | Method | Time |
|-------|--------|------|
| Phase 0: Ground check | CLI + Accessibility API | 1–2 min |
| Phase 1: Preflight | Computer use | 2–3 min |
| Phase 2: Flight | Hybrid | 5–10 min |
| Phase 3: Cleanup | Computer use | 3–5 min |
| Phase 4: Debrief | Text | 2–3 min |
| **Total per tier** | | **13–23 min** |

Full three-tier test: ~45–70 min. Quick Flight (cold only): ~15 min.

---

## Rules

1. **Screenshot every screen before touching it.**
2. **Inventory every element before choosing what to tap.**
3. **Write predictions BEFORE acting.** The mismatch IS the finding.
4. **One action at a time.** Evaluate after each.
5. **No feedback after an action = UX bug.** Always.
6. **Don't skip cleanup.** Untested elements might be broken.
7. **Be honest about confusion.** That's the data.
8. **Reset between tier runs.** Each tier gets a fresh experience.

---

## Smart Retest Scope

After bug fixes, use `git diff` to determine which dimensions need retesting. Skip everything else.

| Change Type | Retest Dimensions | Skip |
|-------------|-------------------|------|
| CSS / layout / styles | Visual Design only | Functional, Navigation, Usability |
| Labels / strings / copy | Usability, Accessibility | Functional, Visual (unless layout changed) |
| Event handlers / logic | Functional Correctness | Visual (unless UI changed) |
| Navigation / routing | Navigation, Functional | Visual (unless layout changed) |
| Data model / API | Functional, Data Integrity | Visual, Usability (unless UI changed) |
| Full feature addition | All dimensions | — |

```bash
# Determine retest scope from the last fix commit
git diff --name-only HEAD~1 | while read f; do
  case "$f" in
    *.css|*.scss|*.styled.*) echo "→ Retest: Visual Design" ;;
    *.strings|*.json|*.md)   echo "→ Retest: Usability + Accessibility" ;;
    *.swift|*.ts|*.py)       echo "→ Retest: Functional Correctness" ;;
  esac
done
```

Full retest: ~20 min. Scoped retest: ~3–5 min. Use scoped retest for single-bug fixes. Use full retest for feature additions or multi-file changes spanning more than one category.

---

*Test Flight Protocol v1.1 — Part of the Test Pilot Loop by Huan Su.*
