# 📊 Flight Log — UX Testing Phase — Knowledge-Tiered Agent Testing
## After Functional Testing Passes, Run Sequential Evaluation Passes With Different Knowledge Levels
**The test that answers: "Can a real person actually use this?"**

---

## 🎯 The Core Insight

The developer knows every button. The product manager knows the PRD. The user knows **nothing**.

After functional testing confirms the app works correctly, **one Opus orchestrator runs three sequential evaluation passes**, each constrained to a different level of knowledge about the project. All three passes use the same app and try to accomplish the same tasks. But each pass starts with a different knowledge constraint — just like real users with varying familiarity. The orchestrator resets to the same starting state between passes.

**The gap between their experiences IS the UX feedback.**

---

## 👥 THE THREE KNOWLEDGE TIERS

### 🔴 Tier 1: "Cold User" — Knows Nothing
```
KNOWLEDGE GIVEN:     App name + icon only. Nothing else.
SIMULATES:           User who just downloaded from App Store
DOES NOT HAVE:       PRD, user manual, tooltips explanation, onboarding context
CORE QUESTION:       "Can someone figure out what this app does and how
                      to use it with ZERO guidance?"

WHAT THIS PASS TESTS:
  • First impressions — what does the launch screen communicate?
  • Discoverability — can they find the main feature?
  • Self-explanatory UI — do labels, icons, and layout make sense alone?
  • Onboarding — does the app guide a new user, or abandon them?
  • Learnability — how long until they can complete a basic task?
  • Frustration points — where do they give up?
  • Mental model — does the app match how they THINK it should work?

METRICS CAPTURED:
  • Steps to First Action (STFA) — agent-steps until first meaningful tap
  • Steps to Task Completion (STTC) — agent-steps to complete core workflow
  • Task Success Rate (TSR) — did they complete it at all? (0 or 1)
  • Error Count — wrong taps, wrong screens, backtracking
  • Wrong Path Count — actions that didn't contribute to completion
  • Abandonment Point — where they would give up in real life
  • Confidence Score — "How sure am I that I did this correctly?"
  • Lostness Score — how many unnecessary screens visited
  • Step Overhead Ratio — their steps vs. minimum possible steps

PERSONA OVERLAY:
  Tier 1 agent also rotates through personas:
  • Run 1: As "Everyday User" (Jordan) — medium patience
  • Run 2: As "Struggling User" (Pat) — reads everything, afraid to tap
  • Run 3: As "Impatient User" (new) — gives up after 10 seconds of confusion
```

### 🟡 Tier 2: "Guided User" — Has the User Manual
```
KNOWLEDGE GIVEN:     App name + User Manual / Help Documentation / README
                     (whatever documentation ships with the product)
DOES NOT HAVE:       PRD, internal architecture, developer notes
SIMULATES:           User who reads the manual before using the app
CORE QUESTION:       "If someone reads your documentation, can they
                      successfully use the product?"

WHAT THIS PASS TESTS:
  • Documentation quality — is the manual accurate and complete?
  • Instruction followability — can they do what the manual says?
  • Gap detection — features in the app not covered by docs
  • Terminology match — does the manual use the same terms as the UI?
  • Screenshot currency — do manual screenshots match current UI?
  • Missing steps — does the manual skip steps that aren't obvious?
  • Error recovery — when they deviate from the manual, can they get back?

METRICS CAPTURED:
  • Manual-to-App Navigation Time — how long to find what the manual describes
  • Documentation Accuracy Rate — % of manual instructions that match reality
  • Documentation Completeness — % of app features covered by the manual
  • Terminology Mismatch Count — manual says X, app shows Y
  • Steps-With-Manual vs Steps-Without — is the manual actually helping?
  • Confidence Score — higher than Tier 1? If not, manual is failing

PERSONA OVERLAY:
  Tier 2 agent tests as:
  • Run 1: "Careful Reader" — follows manual step by step
  • Run 2: "Skimmer" — glances at manual, then tries the app
```

### 🟢 Tier 3: "Insider" — Knows the PRD
```
KNOWLEDGE GIVEN:     PRD.md — full product requirements and acceptance criteria
                     Knows every intended feature, expected behavior, edge case
DOES NOT HAVE:       Source code access (tests as a user, not a developer)
SIMULATES:           Product manager, QA lead, or beta tester with full context
CORE QUESTION:       "Does the app deliver what was specified? Is anything
                      missing, wrong, or different from the requirements?"

WHAT THIS PASS TESTS:
  • Spec compliance — every PRD requirement, verified one by one
  • Feature completeness — all specified features present and working
  • Behavior accuracy — does the app behave exactly as the PRD describes?
  • Edge cases from PRD — specific scenarios called out in requirements
  • Acceptance criteria — does each feature meet its success criteria?
  • Regression — features that worked before but broke in this build
  • UX gaps in the spec — "the PRD said X, but having used it, Y would be better"

METRICS CAPTURED:
  • PRD Compliance Rate — % of requirements fully met
  • Feature Completion Rate — % of specified features implemented
  • Deviation Count — behaviors that differ from PRD spec
  • PRD Gap Detection — requirements that are ambiguous or incomplete
  • Suggested PRD Updates — improvements discovered through actual usage
  • Acceptance Criteria Pass Rate — % of criteria met

PERSONA OVERLAY:
  Tier 3 agent tests as:
  • Run 1: "Strict QA" — checks every requirement literally
  • Run 2: "Product Manager" — evaluates whether the spec was right
```

---

## ⏱️ KEY UX METRICS — What We Measure Across All Tiers

### ⚠️ IMPORTANT: AI Time ≠ Human Time

There is **no established formula** to convert AI agent testing time to human testing time. An AI agent clicks faster, reads faster, and navigates faster than any human. Absolute seconds are not meaningful for predicting human performance.

**What IS meaningful:**

1. **Ratios between tiers** — If Tier 1 takes 5x more steps than Tier 3, that ratio likely holds for humans. A wide gap = your app depends on prior knowledge.
2. **Trends across builds** — If Tier 1 needed 45 steps in Build 1 and 12 steps in Build 4, UX improved 3.7x. The improvement is real regardless of human speed.
3. **Steps and errors, not seconds** — Count taps, wrong screens, backtracking, confusion moments. These are identical for AI and humans. A human also taps 7 times and visits 3 wrong screens.
4. **Binary outcomes** — "Did Tier 1 complete the task: yes or no?" This is the same for AI and human.

**Use "agent-steps" as the unit, not seconds.** One agent-step = one discrete action (tap, scroll, navigate, type, read). This maps directly to human actions.

Research supports this approach: a 2025 study on synthetic heuristic evaluation found that AI evaluations identified 73% and 77% of usability issues across two apps — exceeding the 57% and 63% found by five experienced human evaluators (arXiv:2507.02306). The study also noted limitations with some UI components and cross-screen flows, so AI evaluation complements rather than replaces human testing — but the issue detection rate is strong enough to be useful as a continuous feedback mechanism.

### Effectiveness Metrics (Can they do it?)
| Metric | Definition | How Measured |
|--------|-----------|--------------|
| Task Success Rate (TSR) | % of core tasks completed successfully | Binary: 0 (fail) or 1 (pass) per task |
| Error Rate | Wrong actions per task | Count of wrong taps, wrong screens, backtracking |
| Findability Rate | Can the user locate the feature? | Time to first correct tap on target feature |
| Completion Rate | % of users who finish without abandoning | Did the agent complete, or would they "give up"? |

### Efficiency Metrics (How much effort does it take?)
| Metric | Definition | Unit |
|--------|-----------|------|
| Steps to First Action (STFA) | Agent-steps from app launch to first meaningful interaction | Agent-steps |
| Steps to Task Completion (STTC) | Agent-steps to complete core workflow | Agent-steps |
| Minimum Steps Possible | Fewest steps an expert would need | Agent-steps (baseline) |
| Step Overhead Ratio | STTC / Minimum Steps Possible | Ratio (1.0 = perfect, 3.0 = 3x more effort than needed) |
| Lostness Score | (Unique screens visited / Min screens needed) × (Total screens / Unique screens) | Ratio (1.0 = not lost, > 2.0 = very lost) |
| Backtrack Count | How many times the agent went "back" or revisited a screen | Count |
| Wrong Path Count | Taps/actions that didn't contribute to task completion | Count |

### Satisfaction Metrics (How do they feel about it?)
| Metric | Definition | Scale |
|--------|-----------|-------|
| Single Ease Question (SEQ) | "How easy was this task?" | 1-7 (1=very difficult, 7=very easy) |
| Confidence Score | "How sure are you that you succeeded?" | 1-7 |
| Net Promoter Intent | "Would you recommend this app?" | 1-10 |
| Frustration Points | Moments where the agent would feel frustrated | Count + description of each moment |

### Learnability Metrics (Do they get better?)
| Metric | Definition | How Measured |
|--------|-----------|--------------|
| First-Use Time | TTTC on first attempt | Baseline measurement |
| Second-Use Time | TTTC on second attempt at same task | Should be significantly faster |
| Learning Curve Ratio | First-Use Time / Second-Use Time | > 2.0 means steep learning curve (bad). ~1.0 means already intuitive. |
| Help Dependency | How often did the agent need external help? | Count of "I don't know what to do" moments |

---

## 🔄 UX TEST SESSION WORKFLOW

> **For the screen-by-screen execution protocol** — how the test pilot inspects, inventories, predicts, acts, and evaluates on each screen — see [TEST_FLIGHT_PROTOCOL.md](TEST_FLIGHT_PROTOCOL.md). This section covers the knowledge-tier deployment and comparative analysis workflow.

### Prerequisites
- ✅ Functional testing PASSED (all features work correctly)
- ✅ App is running and accessible via computer use
- ✅ PRD.md and User Manual (if exists) are available
- ✅ Core task list is defined (what should a user be able to do?)

### Phase 1: Define Core Tasks (Cowork Opus)
Before deploying testers, Cowork Opus defines 3-5 core tasks that represent the primary user workflows:

```
CORE TASK LIST FOR UX TESTING
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Project: [Your Project Name]

Task 1: [e.g., "Import a photo album"]
  Start: App launch screen
  End:   Album saved and visible in library
  Minimum steps: [X]

Task 2: [e.g., "Export album as PDF"]
  Start: Library screen
  End:   File exported to chosen destination
  Minimum steps: [X]

Task 3: [e.g., "Compare two albums side by side"]
  Start: Library with 2+ albums
  End:   Comparison view displayed
  Minimum steps: [X]
```

### Phase 2: Deploy Three Agents Sequentially

**How to spawn the tier agents (sequentially — one at a time):**
```
Each tier agent operates the app individually through computer use.
Reset the app to the same starting state before each run.

Run 1: @test-pilot-tier1-cold — gets NO documentation (app name only)
  → Complete core tasks → capture report → reset app

Run 2: @test-pilot-tier2-guided — gets user manual at [path/to/manual]
  → Complete core tasks → capture report → reset app

Run 3: @test-pilot-tier3-insider — gets PRD.md at [path/to/PRD.md]
  → Complete core tasks → capture report

Compare all 3 reports. The gap between them IS the UX feedback.
```

```
┌───────────────────────────────────────────────────────────┐
│                    COWORK OPUS (Supervisor)                │
│        Watches all three agents. Does NOT intervene.       │
│        Records observations. Compiles final report.        │
└──────┬──────────────────┬──────────────────┬──────────────┘
       │                  │                  │
       ▼                  ▼                  ▼
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│ 🔴 TIER 1    │  │ 🟡 TIER 2    │  │ 🟢 TIER 3    │
│ Cold User    │  │ Guided User  │  │ Insider      │
│ @tier1-cold  │  │ @tier2-guided│  │ @tier3-insider│
│              │  │              │  │              │
│ Knows:       │  │ Knows:       │  │ Knows:       │
│ App name     │  │ App name     │  │ App name     │
│ only         │  │ + Manual     │  │ + PRD        │
│              │  │              │  │              │
│ Tries same   │  │ Tries same   │  │ Tries same   │
│ 3-5 tasks    │  │ 3-5 tasks    │  │ 3-5 tasks    │
│              │  │              │  │              │
│ Records:     │  │ Records:     │  │ Records:     │
│ Steps, errors│  │ Steps, errors│  │ Compliance,  │
│ confusion    │  │ doc gaps     │  │ spec gaps    │
└──────────────┘  └──────────────┘  └──────────────┘
```

### Phase 3: Each Agent Tests + Records

Each agent works through the core task list independently, recording every metric. They DO NOT communicate with each other during testing. Each agent uses the structured report template from its subagent file.

### Phase 4: Comparative Analysis (Cowork Opus)

Opus reads all three reports and produces the **UX Gap Analysis**:

```
UX GAP ANALYSIS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Task 1: "Import a photo album"
Minimum steps possible: 4

                    Tier 1      Tier 2      Tier 3      Gap
                    (Cold)      (Manual)    (Insider)   Analysis
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Completed?          ❌ No       ✅ Yes      ✅ Yes      ⚠️ Cold user
                                                        can't complete
Steps taken:        12          7           4           Cold user 3x
                                                        more effort
Step Overhead:      3.0x        1.75x       1.0x        Gap = 3.0x
Wrong paths:        5           2           0           Significant
                                                        confusion
Backtrack count:    3           1           0           Cold user
                                                        wandering
Confidence:         2/7         5/7         7/7         Cold user
                                                        unsure
Lostness:           3.4         1.6         1.0         Cold user
                                                        very lost
SEQ (ease):         2/7         5/7         6/7
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

TIER GAP RATIO: Tier 1 uses 3.0x the steps of Tier 3.
This ratio is meaningful regardless of AI vs. human speed.
A human Tier 1 user would also take ~3x more effort than an expert.

KEY INSIGHT: Cold user couldn't find the import button.
The icon looks like a folder (📁) but the feature is
called "Batch Ingest" — mismatch between icon and label.

RECOMMENDATION: Rename to "Import Photos" + add onboarding tooltip
pointing to the button on first launch.

DOCUMENTATION FINDING: Manual says "tap the import icon on the
main screen" but the icon is actually on the second tab, not
the main screen. Manual is WRONG.

PRD FINDING: PRD doesn't specify onboarding flow. This should
be added as a requirement.
```

---

## 📊 TRACKING OVER TIME — The UX Dashboard

After every build, the UX scores go into your project's persistent memory (e.g., an Obsidian vault or a `ux-dashboard.md` file). Over time, this creates a dashboard:

```
PROJECT: YourApp — UX Trend Dashboard
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Task 1: "Import a photo album" (Minimum steps: 4)

TIER 1 (Cold User) — The True UX Indicator
                  Build 1   Build 2   Build 3   Build 4
Completed?        ❌        ❌        ✅        ✅        📈
Steps taken:      —         —         18        8         📈
Step Overhead:    —         —         4.5x      2.0x      📈
Wrong paths:      5         4         2         1         📈
Confidence:       2/7       3/7       5/7       6/7       📈
SEQ (ease):       2/7       3/7       5/7       6/7       📈

TIER 2 (Manual User) — Documentation Quality Indicator
                  Build 1   Build 2   Build 3   Build 4
Completed?        ✅        ✅        ✅        ✅
Steps taken:      9         7         5         5         📈
Doc Accuracy:     70%       85%       95%       100%      📈

TIER 3 (Insider) — Spec Compliance Indicator
                  Build 1   Build 2   Build 3   Build 4
Steps taken:      4         4         4         4         → (baseline)
PRD Compliance:   60%       75%       90%       95%       📈

TIER GAP RATIO (Tier 1 steps / Tier 3 steps):
                  Build 1   Build 2   Build 3   Build 4
                  —         —         4.5x      2.0x      📈 closing!

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

MILESTONE: Build 3 was the first build where a Cold User
could complete the core task. This is the "usability threshold."

TARGET: Tier Gap Ratio ≤ 1.5x with Tier 1 confidence ≥ 5/7
CURRENT: 2.0x ratio, confidence 6/7 — almost there.

NOTE ON METRICS: All measurements use agent-steps (discrete
actions), not seconds. Ratios and trends are valid indicators
of UX quality regardless of AI vs. human speed differences.
```

---

## 🎯 UX QUALITY GATES — When Is the UX Good Enough?

| Gate | Criteria | What It Means |
|------|----------|---------------|
| 🔴 UX Fail | Tier 1 cannot complete ANY core task | App is not usable. Major UX redesign needed. |
| 🟠 UX Poor | Tier 1 completes < 50% of core tasks | Significant confusion. Needs onboarding and UI clarity work. |
| 🟡 UX Acceptable | Tier 1 completes ≥ 50% with Tier Gap Ratio ≤ 3.0x (Tier 2 completes 100%) | Usable with documentation. Ship if manual is included. |
| 🟢 UX Good | Tier 1 completes ≥ 80%, SEQ ≥ 5/7, Tier Gap Ratio ≤ 2.0x | Most users can figure it out. Ship confidently. |
| ⭐ UX Excellent | Tier 1 completes 100%, confidence ≥ 6/7, Tier Gap Ratio ≤ 1.5x | Intuitive. No manual needed. This is the goal. |

### Human Escalation Based on UX Gates

| Gate | Action |
|------|--------|
| 🔴 UX Fail | **ESCALATE TO HUMAN DIRECTOR.** AI agents cannot fix fundamental UX. Needs human design thinking. |
| 🟠 UX Poor | **Cowork Opus can iterate** — suggest specific UI changes, retest. Escalate to human director if no improvement after 2 iterations. |
| 🟡 UX Acceptable | **Cowork Opus decides** — ship with documentation, or invest in UI improvements. May ask human director for business call. |
| 🟢/⭐ UX Good/Excellent | **Auto-approve.** Ship it. Log the scores. Celebrate. |

---

## 🔁 THE FULL TESTING PIPELINE

```
BUILD COMPLETE
     ↓
┌─────────────────────────────┐
│ PHASE 1: FUNCTIONAL TESTING │ ← Does the code work?
│ (Pilot Handbook)       │
│ Persona-based, 6 dimensions │
│ Severity/Priority bugs       │
└─────────────┬───────────────┘
              │
         PASS? ──── NO → Fix bugs → Retest
              │
             YES
              ↓
┌─────────────────────────────┐
│ PHASE 2: UX TESTING          │ ← Can a human use it?
│ (This document)              │
│ 3 knowledge-tiered agents    │
│ Time-to-task, learnability   │
│ UX Gap Analysis              │
└─────────────┬───────────────┘
              │
      UX Gate? ──── 🔴 FAIL → Escalate to human director
              │         🟠 POOR → Cowork iterates
              │
         🟡+ ACCEPTABLE OR BETTER
              ↓
┌─────────────────────────────┐
│ PHASE 3: SHIP                │
│ Update Obsidian vault        │
│ Log UX scores for trending   │
│ Archive test reports          │
└─────────────────────────────┘
```

---

## 💡 WHY THIS WORKS

Existing approaches each miss something:

**Synthetic persona tools** test mockups with synthetic personas. They never run the real app.

**Unit tests, CI/CD** verify code correctness. They never test whether a human can use it.

**Manual QA testers** test the real app, but they already know the product. They can't un-know what they know.

**This framework** deploys AI agents that genuinely don't know the product (Tier 1 is never given the PRD). They encounter the app the same way a real user would — confused, curious, impatient. And we measure exactly how long it takes them to figure it out.

The gap between Tier 1 and Tier 3 is a **quantitative measure of your UX quality.** When that gap shrinks to near-zero, your app is intuitive. When it's wide, your app depends on documentation or prior knowledge — and most users have neither.

---

*UX Testing Phase Protocol v0.1.0*
*Part of the Test Pilot Loop — Flight Deck by Huan Su.*
*One orchestrator. Three knowledge levels. One truth about your UX.*
