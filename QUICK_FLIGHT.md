# 🛫 Quick Flight
### Three Models, Three Perspectives — Zero Setup
### Opus · Sonnet · Haiku each test your app independently.

---

## ⚠️ Quick Flight is Manual

Quick Flight is a **manual test procedure** — you run each prompt yourself and interpret the results. No Cowork orchestration, no automation loop, no agent coordination. This is how you learn the framework, validate that personas produce meaningful differences, and build trust before engaging the full automated loop.

For the continuous automated cycle, see [TEST_PILOT_LOOP.md](TEST_PILOT_LOOP.md).

---

## What This Is

After your code agent builds a feature, three Claude models **use the app through computer use** — each one independently, each one giving honest human-like feedback. Not just finding bugs. Finding confusion, frustration, and friction.

**The app isn't done when code passes. It's done when all three models can use it.**

---

## Why Three Models

Each Claude model thinks differently. That difference IS the testing:

| Model | Tests As | What It Catches |
|-------|---------|-----------------|
| **Opus** | Thoughtful user | Design flaws, inconsistent patterns, edge cases in flows, subtle UX problems |
| **Sonnet** | Average user | Confusing labels, missing feedback after actions, broken happy path, unclear next steps |
| **Haiku** | Quick user | Obvious breaks, unclear purpose, missing basic functionality, "I have no idea what this does" |

You don't need persona files. Each model's processing depth serves as a proxy for different tester perspectives.

- Opus processes deeply — catches design flaws a careful evaluator would flag
- Sonnet follows the flow — catches what a typical evaluation pass surfaces
- Haiku processes quickly — catches what a shallow first-impression scan reveals

---

## The Loop

```
┌─────────────────────────────────────────────┐
│         CODE AGENT builds feature            │
└──────────────────┬──────────────────────────┘
                   ↓
  🟣 OPUS test-drives via computer use → Report A
     (user switches to next model)
  🔵 SONNET test-drives via computer use → Report B
     (user switches to next model)
  🟡 HAIKU test-drives via computer use → Report C
                   ↓
┌─────────────────────────────────────────────┐
│         AGGREGATE three reports              │
│                                              │
│   3/3 agree → FIX FIRST                     │
│   2/3 agree → FIX NEXT                      │
│   1/3 found → EVALUATE                      │
└──────────────────┬──────────────────────────┘
                   ↓
┌─────────────────────────────────────────────┐
│         CODE AGENT fixes                     │
└──────────────────┬──────────────────────────┘
                   ↓
            ALL THREE APPROVE?
             /            \
           NO              YES
           ↓                ↓
      LOOP AGAIN        ✅ SHIP IT
```

---

## How to Run

### Step 1: Code Agent Builds
Your code agent (Claude Code or any builder) completes the feature. Code compiles. Tests pass.

### Step 2: Launch Your App

The Test Pilot uses your app through **computer use** — it sees your screen and interacts with what's visible. Your app must be running:

| Your Project | How to Launch |
|---|---|
| **macOS app** | Build and run in Xcode |
| **iOS app** | Run in Xcode Simulator |
| **Web app** | Start dev server, open in browser (e.g., `http://localhost:3000`) |
| **Website** | Open the URL in Safari/Chrome |
| **Android app** | Run in Android Studio emulator |
| **CLI tool** | Open Terminal |

The app must be on the **same Mac** where Cowork is running and **visible on screen** (not minimized).

### Step 3: Test With Three Models

Each run is a separate Cowork session. Reset the app to the same starting state between runs.

```
Run 1: Open Cowork with Opus
  → Give it the test prompt (below)
  → Opus test-drives the app via computer use
  → Save Report A
  → Close session

Run 2: Open Cowork with Sonnet
  → Same test prompt, fresh app state
  → Sonnet test-drives the app
  → Save Report B
  → Close session

Run 3: Open Cowork with Haiku
  → Same test prompt, fresh app state
  → Haiku test-drives the app
  → Save Report C
  → Close session
```

Or in Claude Code (code-level feedback, not live UI):
```
/model opus   → run test prompt → save report
/model sonnet → run test prompt → save report
/model haiku  → run test prompt → save report
```

### Step 4: Aggregate Reports

Read all three. Use the agreement rule:

- **3/3 agree** → Definitely a problem. Every type of user will hit it. Fix immediately.
- **2/3 agree** → Probably a problem. Most users will hit it. Fix next.
- **1/3 found it** → Evaluate based on which model:
  - Only Opus → Design subtlety worth considering
  - Only Sonnet → Mainstream issue, probably real
  - Only Haiku → Surface confusion, quick fix or ignore

### Step 5: Feed Back to Code Agent

```
Three test pilots used the feature you built. Here's their feedback:

ALL THREE FOUND:
• [issue 1]
• [issue 2]

TWO FOUND:
• [issue 3]

ONE FOUND:
• [issue 4 — Opus only, evaluate]

Fix the agreed issues first. Then test pilots will retest.
```

### Step 6: Retest

After fixes, run the three models again. Loop until all three report PASS.

---

## The Test Prompt

Give this to each model before testing:

```
You are test-piloting this app through computer use. You are NOT a developer.
You are NOT looking at code. You are interacting with the app through the screen.

THE APP: [how to find it on screen — e.g., "The app is the window titled 
PhotoSortVision" or "Open Safari and go to http://localhost:3000" or 
"The iOS Simulator is running on the right side of the screen"]

A new feature was just built: [one sentence describing what was built]

Your task: [what a user would try to do with this feature]

Use the app through computer use. Think out loud as you go.
After testing, fill out the report below.

Be honest. If something confuses you, say so. If you don't know
what a button does, say so. If the app doesn't give you feedback
after an action, say so. Your confusion IS the feedback.
```

---

## The Report Template

Each model fills this out after testing:

```
TEST PILOT REPORT
═════════════════
Model: [Opus / Sonnet / Haiku]
Feature tested: [what was built]
Task attempted: [what they tried to do]

COMPLETED: YES / NO / PARTIALLY
If NO: where I got stuck and why

STEPS TAKEN: [count]
WRONG TURNS: [count]

────────────────────────────────────
WHAT WORKED WELL
────────────────────────────────────
• [specific thing that felt clear and natural]
• [specific thing that felt clear and natural]

────────────────────────────────────
WHAT CONFUSED ME
────────────────────────────────────
• [where, what happened, what I expected instead]
• [where, what happened, what I expected instead]

────────────────────────────────────
WHAT I'D CHANGE
────────────────────────────────────
• [specific suggestion and why]
• [specific suggestion and why]

────────────────────────────────────
BUGS FOUND
────────────────────────────────────
• [what happened vs what should have happened]

────────────────────────────────────
INTERFACE EVALUATION
────────────────────────────────────
Layout clear:         YES / MOSTLY / NO — [explain]
Labels make sense:    YES / MOSTLY / NO — [which ones don't]
Feedback after taps:  YES / SOMETIMES / NEVER — [where missing]
Know what to do next: YES / SOMETIMES / NO — [where lost]
Would use this app:   YES / MAYBE / NO — [why]

────────────────────────────────────
ONE-SENTENCE SUMMARY
────────────────────────────────────
"[The single most important thing to fix before shipping.]"
```

---

## Aggregated Feedback Template

After all three tests:

```
AGGREGATED TEST PILOT FEEDBACK
═══════════════════════════════
Feature: [what was built]

ALL THREE AGREE (fix immediately):
  • [issue]

TWO OF THREE AGREE (fix next):
  • [issue — which two found it]

ONLY ONE FOUND (evaluate):
  • Opus only: [issue]
  • Sonnet only: [issue]
  • Haiku only: [issue]

BUGS:
  • [any broken functionality]

CONSENSUS:
  Opus:   PASS / NEEDS WORK / FAIL
  Sonnet: PASS / NEEDS WORK / FAIL
  Haiku:  PASS / NEEDS WORK / FAIL

  → ALL PASS = Ship it ✅
  → ANY FAIL = Fix and reloop ❌
  → NEEDS WORK only = Fix agreed items, retest
```

---

## Quick Mode (Don't Always Need All Three)

| Situation | Use |
|-----------|-----|
| Quick check after small fix | **Haiku only** — 2 min, catches obvious breaks |
| Standard feature validation | **Sonnet only** — 5 min, catches mainstream UX |
| Critical feature or milestone | **All three** — 15 min, comprehensive |
| Design/architecture review | **Opus only** — deep thinking about flow |

---

## What This Catches That Code Tests Don't

Code tests verify: *does the function return the right value?*

Test pilots verify:
- "I can't find the button" → discoverability
- "I tapped it but nothing happened" → missing feedback
- "I don't know if it worked" → unclear success states
- "This label doesn't mean what I think" → confusing terminology
- "I expected it to go here but it went there" → broken mental model
- "There's no way to undo this" → safety
- "I don't know what to do next" → missing guidance
- "This takes too many steps" → workflow friction

**These are the problems that make users uninstall.** Code tests will never catch them.

---

## Scaling Up

When you're ready for more depth, see the **Test Pilot Loop** (one model, three knowledge levels, continuous loop) or the **full framework** in the FLIGHT_DECK/ folder with personas, archetypes, risk tiers, and orchestration.

---

*Quick Flight v0.1.0*
*Three models. Three perspectives. Real feedback.*
*Part of the Test Pilot Loop Framework by Huan Su.*
