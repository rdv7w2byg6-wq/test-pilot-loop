# 📋 Pilot Handbook — Test Pilot Protocol v0.1.0
## Live, Persona-Based UX Testing Through Computer Use
**A framework where an AI agent tests software through computer use — clicking, typing, and navigating — with different personas, UX evaluation, accessibility checks, and instant feedback integrated into the build loop.**

---

## 🎯 What This Is

Traditional testing asks: **"Does the button work?"**
This protocol asks: **"Would a real person understand what to do, enjoy doing it, and succeed — even if they're not tech-savvy?"**

The AI Test Pilot (Cowork Opus) uses computer use to:
- Launch and interact with the **live running app** (not mockups, not screenshots)
- Inhabit **different user personas** with varying tech literacy, patience, and expectations
- Evaluate **functionality, usability, visual design, accessibility, and user journey flow**
- Produce **structured feedback** that maps directly to actionable fixes
- Feed results **back into the build loop** for immediate iteration

---

## 📋 PREREQUISITES — What the Test Pilot Needs

Before any test session, the Test Pilot must have:

### 1. Project Understanding
- **PRD.md** — what the app is supposed to do (product requirements)
- **Target user profiles** — who will use this software
- **Success criteria** — what "working correctly" looks like
- **Platform context** — iOS simulator? macOS app? Web browser? Remote desktop?

### 2. App Access
- Running instance of the app (simulator, localhost, deployed build)
- Cowork folder mounts to project directory
- Screenshot save location (`AUDIT_SCREENSHOTS/`)

### 3. Test Scope (from orchestration file)
- Which task(s) just completed
- What feature was built or changed
- What files were modified
- Agent's self-assessment and confidence score

---

## 👥 USER PERSONAS — The Test Pilot's Identities

The Test Pilot doesn't test as "an AI." It tests as **specific human personas** with defined psychological profiles, behavioral constraints, and predictable reactions. The specificity of these behaviors is what makes the testing produce different results per persona — not just different labels.

> **📁 Canonical source:** The full persona definitions live in `PERSONAS/agents/`. Those files are the source of truth. This section is a summary.

### Core 4 Personas (Use for Every Project)

| Persona | File | Mindset | Key Behavior | Tests For |
|---------|------|---------|-------------|-----------|
| 🟢 **Alex** (Power User) | `test-pilot-alex.md` | "Don't slow me down." | Skips all tutorials, hunts for Cmd+shortcuts, tries bulk ops, rage-taps if slow | Efficiency, missing shortcuts, workflow friction |
| 🟡 **Jordan** (Everyday User) | `test-pilot-jordan.md` | "I just want to do the thing." | Follows the biggest button, reads labels not paragraphs, expects feedback after every tap | Confusing labels, broken happy path, missing feedback |
| 🔴 **Pat** (Struggling User) | `test-pilot-pat.md` | "I'm going to break it." | Reads EVERYTHING, afraid of "Execute" button, panics at red text, gives up after 3 confusing steps | Cognitive overload, scary jargon, missing undo, small targets |
| ♿ **Sam** (Accessibility User) | `test-pilot-sam.md` | "Software must adapt to me." | Navigates ONLY by Tab/Enter, checks contrast ratios, focus indicators, screen reader order, touch targets | WCAG compliance, keyboard traps, low contrast, missing alt-text |

### Knowledge Tiers (Use for UX Phase Testing)

| Tier | File | What They Know | What They Test |
|------|------|---------------|----------------|
| 🔴 **Tier 1** (Cold User) | `test-pilot-tier1-cold.md` | App name ONLY | Discoverability, first impressions, learnability |
| 🟡 **Tier 2** (Guided User) | `test-pilot-tier2-guided.md` | User manual only | Documentation accuracy, instruction followability |
| 🟢 **Tier 3** (Insider) | `test-pilot-tier3-insider.md` | Full PRD | Spec compliance, feature completeness, acceptance criteria |

### Universal Archetypes (Layer on Top of Personas)

| Archetype | File | When to Apply | Focus |
|-----------|------|--------------|-------|
| **Creator** | `archetype-overlays.md` | Data entry, import, forms, uploads | Flow state, data safety, validation |
| **Administrator** | `archetype-overlays.md` | Settings, config, permissions, exports | Control, visibility, bulk management |
| **Consumer** | `archetype-overlays.md` | Viewing results, browsing, dashboards | Zero friction, visual polish, instant value |

**How to combine:** Personas describe WHO (skill level). Archetypes describe WHAT (task type). Knowledge tiers describe HOW MUCH they know. Combine all three for the richest testing:
- "Pat (Struggling User) as Creator (importing photos) at Tier 1 (knows nothing)"

### Domain-Specific Personas (Add Your Own)

Create additional `.claude/agents/test-pilot-[name].md` files for your domain. See `PERSONAS/README.md` for the template.

**Examples:** Busy Clinician, Teen User, IT Admin, First-Time Buyer — see persona files for format.

---

## 🔍 TEST DIMENSIONS — What the Test Pilot Evaluates

Every test session evaluates **six dimensions.** The Test Pilot scores each on a 5-point scale.

### Dimension 1: Functional Correctness ⚙️
> "Does it do what it's supposed to do?"

| Check | Description |
|-------|-------------|
| Feature works | The built feature performs its intended function |
| Input handling | Accepts valid input, rejects invalid input gracefully |
| State management | App maintains correct state across navigation |
| Error handling | Errors produce clear messages, not crashes |
| Data persistence | Saved data is still there after navigating away and back |

**Score:** 1 (Broken) · 2 (Mostly broken) · 3 (Works with issues) · 4 (Works well) · 5 (Flawless)

### Dimension 2: Usability & Clarity 🧭
> "Can the user figure out what to do without help?"

| Check | Description |
|-------|-------------|
| Discoverability | Can the user find the feature? Is the entry point obvious? |
| Labeling | Are buttons, fields, and sections labeled clearly? No jargon? |
| Feedback | Does the app tell the user what happened after each action? |
| Error recovery | When something goes wrong, can the user recover easily? |
| Cognitive load | Is the user asked to remember or process too much at once? |
| Flow logic | Does the step sequence make sense? Any unnecessary steps? |

**Score:** 1 (Confusing) · 2 (Unclear) · 3 (Usable with effort) · 4 (Clear) · 5 (Intuitive)

### Dimension 3: Visual Design & Layout 🎨
> "Does it look right? Is the layout professional and consistent?"

| Check | Description |
|-------|-------------|
| Alignment | Are elements aligned on a grid? Any misaligned items? |
| Spacing | Consistent padding/margins? Nothing crowded or floating? |
| Typography | Font sizes appropriate? Hierarchy clear? Readable? |
| Color usage | Colors consistent? Good contrast? Meaningful use of color? |
| Visual hierarchy | Does the eye flow naturally to the most important element first? |
| Platform conventions | Does it follow iOS/macOS/Android/web design conventions? |
| Empty states | What does it look like with no data? Helpful or blank? |

**Score:** 1 (Broken layout) · 2 (Messy) · 3 (Acceptable) · 4 (Clean) · 5 (Polished)

### Dimension 4: Accessibility ♿
> "Can everyone use it?"

| Check | Description |
|-------|-------------|
| Color contrast | Minimum 4.5:1 for text, 3:1 for large text (WCAG AA) |
| Touch targets | Minimum 44x44pt (iOS) or 48x48dp (Android) |
| Dynamic type | Does the UI adapt to system font size settings? |
| Screen reader | Can VoiceOver/TalkBack navigate the feature? Labels present? |
| Motion | Animations respect reduced-motion settings? |
| Single-hand use | Can core features be used with one hand? |

**Score:** 1 (Inaccessible) · 2 (Major barriers) · 3 (Partially accessible) · 4 (Mostly accessible) · 5 (Fully accessible)

### Dimension 5: User Journey Completion 🗺️
> "Can the user complete the full intended workflow from start to finish?"

| Check | Description |
|-------|-------------|
| End-to-end flow | Can the user start from the entry point and reach the goal? |
| No dead ends | Does every screen have a clear next step or exit? |
| Progress indication | Does the user know where they are in the process? |
| Interruption recovery | If the user leaves mid-flow, can they resume? |
| Success confirmation | When finished, does the user know they succeeded? |

**Score:** 1 (Cannot complete) · 2 (Completes with major friction) · 3 (Completes with minor issues) · 4 (Smooth completion) · 5 (Delightful completion)

### Dimension 6: Edge Case Resilience 🔨
> "What happens when the user does something unexpected?"

| Check | Description |
|-------|-------------|
| Empty input | Submit with nothing entered |
| Very long input | Enter extremely long text/numbers |
| Special characters | Enter emoji, Unicode, symbols, HTML tags |
| Rapid interaction | Tap the same button 5 times fast |
| Back navigation | Hit back at every step — does the app handle it? |
| Rotation/resize | Rotate device or resize window mid-flow |
| Network loss | What happens if connection drops? (if applicable) |

**Score:** 1 (Crashes) · 2 (Breaks badly) · 3 (Handles some) · 4 (Handles most) · 5 (Bulletproof)

---

## 🔄 TEST SESSION WORKFLOW

> **For the step-by-step execution protocol** — how to inspect each screen, inventory elements, form predictions, and evaluate actions — see [TEST_FLIGHT_PROTOCOL.md](TEST_FLIGHT_PROTOCOL.md). This section covers the persona rotation and session structure; the protocol covers the screen-by-screen testing cycle (SEE→INVENTORY→PREDICT→ACT→EVALUATE).

### How to Run Persona-Based Testing

> **Canonical mode (default):** Cowork Opus drives the app through computer use and adopts each persona's behavioral rules in sequential passes. **Persona subagents are an optional helper pattern** — they run in Claude Code for code-side evaluation and report generation, not for direct UI interaction.

**Option A: Cowork Opus sequential rotation (default — recommended)**
Opus adopts each persona sequentially, re-reading the persona's behavioral rules before each run. One agent drives the UI through computer use for all passes.
```
Test [feature] sequentially with each persona:
  Run 1: @test-pilot-alex — test as Power User → capture report
  Run 2: @test-pilot-jordan — test as Everyday User → capture report
  Run 3: @test-pilot-pat — test as Struggling User → capture report
  Run 4: @test-pilot-sam — test as Accessibility User → capture report
  
Reset the app to the same starting state before each run.
Each persona reports independently. Compare results after all 4 complete.
```

**Option B: Sequential rotation (Cowork Opus tests as each persona in turn)**
Opus adopts each persona sequentially, re-reading the persona's behavioral rules before each run. Use when computer use requires a single agent operating the UI.

**Option C: Agent Teams (for deep collaborative testing)**
```
Create a test pilot team. Spawn 4 teammates:
  "alex" with prompt from test-pilot-alex.md
  "jordan" with prompt from test-pilot-jordan.md
  "pat" with prompt from test-pilot-pat.md
  "sam" with prompt from test-pilot-sam.md
Have them test [feature] and share findings with each other.
```

### For Each Completed Task (🟡 Medium / 🔴 High Risk):

```
PHASE 1: CONTEXT LOADING (2 min)
  • Read PRD.md — understand what the app should do
  • Read the completed task from AGENT OUTPUT LOG — what was built
  • Read the agent's confidence score and self-assessment
  • Select archetype overlay based on feature type:
    - Data entry / import / form → Creator archetype
    - Settings / config / management → Admin archetype
    - Viewing / browsing / results → Consumer archetype

PHASE 2: FUNCTIONAL TEST (3-5 min)
  • Launch the app via computer use
  • Spawn @test-pilot-alex (or test as Alex persona): core functionality
  • Verify: does it do what the task said it should do?
  • Try happy path, then deliberately try to break it
  • Score Dimension 1 (Functional) and Dimension 6 (Edge Cases)

PHASE 3: PERSONA ROTATION (5-10 min)
  • Spawn @test-pilot-jordan (or switch to Jordan persona)
    - Try to use the feature without any prior knowledge
    - Note where Jordan pauses, hesitates, or feels unsure
    - Score Dimension 2 (Usability)

  • Spawn @test-pilot-pat (or switch to Pat persona)
    - Approach the feature as Pat — afraid to break things
    - Read every label. Try the most obvious-looking action first.
    - Note anything confusing, scary, or unclear
    - Score Dimension 2 (Usability) from Pat's perspective

  • Spawn @test-pilot-sam (or switch to Sam persona)
    - Navigate ONLY with keyboard (Tab, Enter, Escape)
    - Check contrast, touch targets, dynamic type, focus indicators
    - Score Dimension 4 (Accessibility)

PHASE 4: VISUAL & LAYOUT REVIEW (2-3 min)
  • Take a step back — look at the screen holistically
  • Check alignment, spacing, typography, color usage
  • Compare to other screens in the app — is it consistent?
  • Score Dimension 3 (Visual Design)

PHASE 5: USER JOURNEY TEST (3-5 min)
  • As "Everyday User": complete the full workflow from entry to goal
  • Note any dead ends, unclear next steps, missing confirmation
  • Score Dimension 5 (User Journey)

PHASE 6: REPORT (3 min)
  • Write the Test Pilot Report (see template below)
  • Save screenshots for anything notable
  • Write verdict: PASS / CONDITIONAL PASS / FAIL
```

---

## 📝 TEST PILOT REPORT TEMPLATE

```
[YYYY-MM-DD HH:MM] — AI Test Pilot Report
═══════════════════════════════════════════

Task ID: [T00X]
Feature Tested: [description of what was built]
App State: [simulator / localhost / remote desktop / deployed build]
Build Confidence: [agent's self-reported confidence score]

PERSONAS USED:
  • Expert User (Alex): [tested / not tested]
  • Everyday User (Jordan): [tested / not tested]
  • Struggling User (Pat): [tested / not tested]
  • Accessibility User (Sam): [tested / not tested]
  • Domain Persona: [name — tested / not tested]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

DIMENSION SCORES:
  ⚙️ Functional Correctness:    [1-5] / 5
  🧭 Usability & Clarity:       [1-5] / 5
  🎨 Visual Design & Layout:    [1-5] / 5
  ♿ Accessibility:              [1-5] / 5
  🗺️ User Journey Completion:   [1-5] / 5
  🔨 Edge Case Resilience:      [1-5] / 5
  ─────────────────────────────
  OVERALL:                       [X] / 30

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

PERSONA FINDINGS:

🟢 Expert User (Alex):
  • [What worked well from a power-user perspective]
  • [What frustrated or slowed them down]

🟡 Everyday User (Jordan):
  • [What was clear and intuitive]
  • [Where they hesitated or got confused]
  • [Any labels, icons, or flows that were unclear]

🔴 Struggling User (Pat):
  • [Could they complete the task at all?]
  • [Where they would give up in the real world]
  • [What needs explicit guidance or help text]

♿ Accessibility User (Sam):
  • [Contrast issues found]
  • [Touch target problems]
  • [Screen reader gaps]
  • [Dynamic type issues]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

SPECIFIC ISSUES FOUND:

Issue #1:
  Severity: 🔴 Critical / 🟡 Major / 🟢 Minor / ⚪ Cosmetic
  Persona(s) affected: [which personas hit this]
  Screen/Feature: [where it happens]
  Steps to reproduce: [exact steps]
  Expected behavior: [what should happen]
  Actual behavior: [what actually happened]
  Screenshot: [path]
  Suggested fix: [if obvious]

Issue #2:
  [same format]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

UX RECOMMENDATIONS:
  • [Layout suggestion — e.g., "Move the save button above the fold"]
  • [Flow suggestion — e.g., "Add a confirmation step before delete"]
  • [Copy suggestion — e.g., "Change 'Execute' to 'Import Photos'"]
  • [Design suggestion — e.g., "Increase contrast on secondary labels"]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

VERDICT:
  ✅ PASS (Score ≥ 25/30, no 🔴 Critical issues)
  ⚠️ CONDITIONAL PASS (Score 20-24, or has 🟡 Major issues — fix before next task)
  ❌ SOFT FAIL (Score 15-19 — specific fixes needed, retest after)
  ❌ HARD FAIL (Score < 15 or 🔴 Critical — escalate to brain for architectural review)

NEXT STEPS:
  [specific instructions for the build agent — which issues to fix first]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Screenshots saved: [count] files in AUDIT_SCREENSHOTS/
```

---

## 📊 SCORING THRESHOLDS

| Overall Score | Verdict | Action |
|--------------|---------|--------|
| 25-30 | ✅ PASS | Approve task. Proceed to next. |
| 20-24 | ⚠️ CONDITIONAL PASS | Approve with caveats. Log issues for later fix. |
| 15-19 | ❌ SOFT FAIL | Send back to build agent with specific fixes. Retest after. |
| Below 15 | ❌ HARD FAIL | Escalate to Opus brain for architectural review. Something is fundamentally wrong. |

**Override rules:**
- ANY 🔴 Critical issue = automatic FAIL regardless of score
- Accessibility score of 1 or 2 = automatic FAIL (not shipping inaccessible software)
- User Journey score of 1 = automatic FAIL (users can't complete the basic flow)

---

## 🔁 RETEST PROTOCOL

When a task fails and is sent back for fixes:

1. Build agent fixes the specific issues listed in the Test Pilot Report
2. Build agent sets status → `⏳ Awaiting Review`
3. Test Pilot runs an **abbreviated retest** — only the failed dimensions:
   - Verify each reported issue is fixed
   - Re-score only the dimensions that failed
   - Check that fixes didn't break other dimensions (regression)
4. If all issues resolved → ✅ PASS
5. If new issues introduced → ❌ FAIL with new report

**Maximum retest cycles:** 3. If a feature fails 3 times, escalate to Opus brain for redesign.

---

## ⚡ QUICK TEST vs. FULL TEST

Not every task needs a full 6-dimension persona rotation.

| Task Risk Level | Test Type | Duration | Personas | Dimensions |
|----------------|-----------|----------|----------|------------|
| ⚪ Trivial | No test | 0 min | None | Opus patrol fast-approve only |
| 🟢 Low | Quick Test | 2-3 min | Expert only | Functional + Edge Cases |
| 🟡 Medium | Standard Test | 10-15 min | Expert + Everyday + Accessibility | All 6 dimensions |
| 🔴 High | Full Test | 15-25 min | All 4+ domain persona | All 6 dimensions + regression |
| 🏁 Milestone | Comprehensive Test | 30+ min | All personas, full journey, all screens | Full regression suite |

---

## 🎯 REAL HUMAN TESTER AGENDA — What They Focus On

For reference, here is what a professional human QA tester's agenda looks like for a typical test session. The AI Test Pilot should match this level of rigor:

**Pre-Session (5 min):**
- Review what was changed and why
- Understand the expected user flow
- Identify risk areas based on complexity of changes

**Core Testing (20-30 min):**
1. **Smoke test** — does the app launch, does the main feature work at all?
2. **Happy path** — follow the intended user flow start to finish
3. **Negative testing** — enter wrong data, skip steps, go backward
4. **Boundary testing** — minimum/maximum values, empty states, long inputs
5. **Regression check** — did fixing one thing break something else?
6. **Visual inspection** — alignment, spacing, consistency with existing screens
7. **Accessibility quick-check** — contrast, font scaling, touch targets

**Post-Session (5-10 min):**
- Prioritize findings: critical → major → minor → cosmetic
- Write clear reproduction steps for each bug
- Suggest fixes when the solution is obvious
- Flag anything that needs design input vs. just code fixes

**What separates good from great testers:**
- They test the **transitions** (what happens between screens, during loading, after errors)
- They test with **real-world data** (not just "test123" — use realistic, domain-appropriate data that reflects real workflows)
- They test **interruptions** (phone call mid-flow, app backgrounded, low battery warning)
- They notice **inconsistency** (this screen uses "Cancel" but that screen uses "Back")
- They think about **first impressions** (what does a brand-new user see?)

---

## 🔮 FUTURE EXTENSIONS

### Multi-Device Testing
Test the same feature on multiple device sizes/orientations in sequence.

### Competitive Benchmarking
Compare UX against competitor apps: "Is our onboarding faster than CompetitorApp's?"

### Longitudinal Testing
Track dimension scores over time. Is UX improving or degrading with each release?

### A/B Layout Testing
Test two versions of a screen with different personas and compare scores.

### Automated Regression Suite
Build a library of persona-based test scenarios that run after every significant change.

---

*Test Pilot Protocol v0.1.0*
*Part of the Test Pilot Loop — Flight Deck by Huan Su.*
*Persona-based, UX-aware, live app testing through computer use.*
