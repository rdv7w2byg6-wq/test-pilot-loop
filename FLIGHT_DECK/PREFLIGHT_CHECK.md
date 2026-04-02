# ✅ Preflight Check — Test Pilot Configuration
## Customize Your Testing — Everything Is Adjustable
**Lives at `FLIGHT_DECK/PREFLIGHT_CHECK.md` in your project. The Test Pilot reads it before every session.**

---

## 🎯 Why This File Exists

The Test Pilot Loop framework defaults to maximum rigor: 4 personas, 3 knowledge tiers, 6 dimensions, full scoring. That's thorough — but it's also expensive and time-consuming.

**Not every project needs maximum rigor.**

A weekend side project needs a quick sanity check, not a 7-persona deep dive. A medical device app needs the full suite plus domain personas. This file lets you dial the testing up or down based on what your project actually needs.

**Everything below is optional. If this file doesn't exist, the framework uses defaults.**

---

## 👥 PERSONA SELECTION

Choose which personas to deploy. Minimum: 1. Recommended: 2-4.

```
ACTIVE PERSONAS:
  ✅ Alex (Power User)          — deploy: YES / NO
  ✅ Jordan (Everyday User)     — deploy: YES / NO
  ✅ Pat (Struggling User)      — deploy: YES / NO
  ✅ Sam (Accessibility User)   — deploy: YES / NO
```

### Presets

**Quick Test (1 persona — fastest, cheapest)**
Best for: ⚪ Trivial and 🟢 Low risk tasks, rapid iteration, early prototyping
```
  ✅ Jordan (Everyday User)
  ❌ Alex, Pat, Sam — skipped
```
Why Jordan: catches the most common issues for the broadest user base.

**Standard Test (2 personas — good balance)**
Best for: 🟡 Medium risk tasks, most development work
```
  ✅ Jordan (Everyday User)
  ✅ Pat (Struggling User)
  ❌ Alex, Sam — skipped
```
Why these two: Jordan finds mainstream confusion, Pat finds cognitive overload. Together they cover ~90% of real user problems.

**Full Test (4 personas — maximum coverage)**
Best for: 🔴 High risk tasks, milestone reviews, pre-release
```
  ✅ Alex (Power User)
  ✅ Jordan (Everyday User)
  ✅ Pat (Struggling User)
  ✅ Sam (Accessibility User)
```

**Accessibility-First (2 personas — compliance focus)**
Best for: apps with legal accessibility requirements, healthcare, government
```
  ✅ Sam (Accessibility User)
  ✅ Pat (Struggling User)
  ❌ Alex, Jordan — skipped
```

### Custom Personas

Add your own domain personas. Set deploy to YES/NO per session.

```
CUSTOM PERSONAS:
  ⬜ [Name] ([Role])            — deploy: YES / NO
     Subagent file: .claude/agents/test-pilot-[name].md
  ⬜ [Name] ([Role])            — deploy: YES / NO
     Subagent file: .claude/agents/test-pilot-[name].md
```

---

## 🧠 KNOWLEDGE TIER SELECTION

Choose which knowledge tiers to deploy for UX testing. Minimum: 1. Recommended: 2-3.

```
ACTIVE TIERS:
  ✅ Tier 1 — Cold User (knows nothing)     — deploy: YES / NO
  ✅ Tier 2 — Guided User (has manual)       — deploy: YES / NO   ⚠️ REQUIRES user guide/manual/handbook
  ✅ Tier 3 — Insider (has PRD)              — deploy: YES / NO
```

**Deployment prerequisite:** Tier 2 (Guided) requires a user guide, manual, handbook, or help documentation to exist. If there is no documentation to test against, do NOT deploy Guided — it has nothing to work with. Skip it and use Cold + Insider instead.

### Deployment Logic

Each tier does one thing the others can't:

- **Insider** — exhaustive spec compliance. Tests every function, every button, every expected outcome. Catches all bugs. **Always deploy.**
- **Cold** — first impressions and discoverability. Can a stranger figure it out? **Add when you want UX/design feedback.**
- **Guided** — manual accuracy cross-check. Does the documentation match reality? **Add only when a manual exists.**

### Presets

**Quick UX Check (1 tier)**
```
  ✅ Tier 1 — Cold User only
```
Why: if a cold user can complete the task, UX is good enough. Fastest signal.

**Standard UX Test (2 tiers)**
```
  ✅ Tier 1 — Cold User
  ✅ Tier 3 — Insider
```
Why: the gap between these two IS the Tier Gap Ratio. Skipping Tier 2 saves time while still measuring the knowledge gap.

**Insider Only — Exhaustive Spec Audit + Outcome Verification (1 tier)**
```
  ❌ Tier 1 — Cold User — skipped
  ❌ Tier 2 — Guided User — skipped
  ✅ Tier 3 — Insider (exhaustive element audit + outcome verification)
```
Why: the Insider tests every function against the spec AND verifies the actual output (opens Finder, checks files, investigates failures). Uses the Five Lenses: Works? Achieves goal? Hard to use? Makes sense? What else? Catches all functional bugs plus logic issues other tiers would miss. Best for: professional/trained-user apps where users know the workflow (e.g., dental practice software, internal tools, B2B apps). See `FLIGHT_DECK/INSIDER_TESTING_SKILL.md` for the full 8-phase protocol.

**What you trade away:** Cold tier first-impression testing (would a stranger figure this out?) and Guided tier documentation testing (does the manual match reality?). For consumer apps, those matter. For professional tools with trained users, they're noise.

**Optional: Layer UX tiers for design feedback.** Even with Insider-only for bug finding, you can add Cold or Guided tiers occasionally for *design commentary* — layout, visual hierarchy, font size, button placement, color choices. These are not bug reports; they are design inputs for the human director and Cowork to evaluate. Route them separately: bugs go to Claude Code for fixing, design feedback goes to the human for a decision.

**Full UX Test (3 tiers)**
```
  ✅ Tier 1 — Cold User
  ✅ Tier 2 — Guided User
  ✅ Tier 3 — Insider
```
Why: also tests documentation quality (Tier 2). Use for pre-release.

---

## 🔍 DIMENSION SELECTION

Choose which of the 6 test dimensions to evaluate. Minimum: 1. Recommended: 3-6.

```
ACTIVE DIMENSIONS:
  ✅ ⚙️ Functional Correctness    — evaluate: YES / NO
  ✅ 🧭 Usability & Clarity        — evaluate: YES / NO
  ✅ 🎨 Visual Design & Layout     — evaluate: YES / NO
  ✅ ♿ Accessibility               — evaluate: YES / NO
  ✅ 🗺️ User Journey Completion    — evaluate: YES / NO
  ✅ 🔨 Edge Case Resilience       — evaluate: YES / NO
```

### Presets

**Functional Only (1 dimension — fastest)**
```
  ✅ ⚙️ Functional Correctness
  ❌ All others
```
Use for: smoke testing, quick validation after bug fix.

**UX Focus (3 dimensions)**
```
  ✅ 🧭 Usability & Clarity
  ✅ 🗺️ User Journey Completion
  ✅ 🔨 Edge Case Resilience
```
Use for: UX-heavy features, onboarding flows, forms.

**Visual Focus (2 dimensions)**
```
  ✅ 🎨 Visual Design & Layout
  ✅ ♿ Accessibility
```
Use for: UI redesigns, theme changes, responsive layout work.

**Full Evaluation (6 dimensions)**
```
  ✅ All six dimensions
```
Use for: milestone reviews, pre-release, critical features.

---

## 📊 SCORING THRESHOLDS

Adjust pass/fail criteria based on your project's quality bar.

```
PASS THRESHOLDS:
  ✅ PASS score:              [25] / 30    (default: 25)
  ⚠️ CONDITIONAL PASS score:  [20] / 30    (default: 20)
  ❌ SOFT FAIL score:         [15] / 30    (default: 15 — send back for fixes)
  ❌ HARD FAIL score:         below [15]    (default: below 15 — escalate to brain)
```

```
UX QUALITY GATES:
  ⭐ Excellent: Tier Gap Ratio ≤ [1.5]x    (default: 1.5)
  🟢 Good:     Tier Gap Ratio ≤ [2.0]x     (default: 2.0)
  🟡 Acceptable: Tier Gap Ratio ≤ [3.0]x   (default: 3.0)
  🔴 Fail:     Tier 1 cannot complete        (always applies)
```

```
AUTO-FAIL OVERRIDES (always applies regardless of score):
  ✅ Any 🔴 Critical bug = automatic FAIL         — override: YES / NO
  ✅ Accessibility score 1-2 = automatic FAIL      — override: YES / NO
  ✅ User Journey score 1 = automatic FAIL          — override: YES / NO
```

---

## 🏢 ARCHETYPE OVERLAY

Choose which archetype to apply based on what you're testing. Set per feature or per session.

```
DEFAULT ARCHETYPE: [Creator / Admin / Consumer / Auto]
```

**Auto mode:** The Test Pilot selects the archetype based on the feature:
- Data entry / import / form features → Creator
- Settings / config / management → Admin
- Viewing / browsing / results → Consumer

---

## ⏱️ TESTING DEPTH PER RISK LEVEL

Customize how much testing each risk level gets.

```
RISK LEVEL TESTING:

⚪ Trivial:
  Personas: [0]  (default: 0 — Opus patrol fast-approve)
  Tiers: [0]
  Dimensions: [0]

🟢 Low:
  Personas: [1]  (default: 1 — Jordan only)
  Tiers: [0]     (default: 0 — no UX tier testing)
  Dimensions: [2] (default: Functional + Edge Cases)

🟡 Medium:
  Personas: [2-3] (default: Jordan + Pat + optionally Sam)
  Tiers: [2]      (default: Tier 1 + Tier 3)
  Dimensions: [6] (default: all)

🔴 High:
  Personas: [4+]  (default: all 4 + domain personas)
  Tiers: [3]      (default: all 3 tiers)
  Dimensions: [6] (default: all)

🏁 Milestone:
  Personas: [4+]  (default: all + domain)
  Tiers: [3]      (default: all 3)
  Dimensions: [6] (default: all)
  Additional: Full regression + user journey end-to-end
```

---

## 🔄 RETEST CONFIGURATION

```
Max retest cycles before escalation: [3]    (default: 3)
Retest scope: [failed dimensions only / full retest]  (default: failed only)
```

---

## 🧠 MODEL SELECTION FOR TESTING

```
Orchestrator / Test Pilot:  Opus          (always — this is Cowork Opus)
Persona subagents:          [sonnet / opus]    (default: sonnet)
Upgrade subagents to Opus:  [🔴 High risk only / 🟡+ / always]  (default: 🔴 only)
```

---

## 📝 EXAMPLE CONFIGURATIONS

### Weekend Side Project
```
Personas: Jordan only
Tiers: Tier 1 only
Dimensions: Functional + Usability
Pass threshold: 15/30
Persona model: sonnet
```
Total testing time per feature: ~5 minutes

### Production SaaS App
```
Personas: Jordan + Pat + Sam
Tiers: Tier 1 + Tier 3
Dimensions: All 6
Pass threshold: 22/30
Persona model: sonnet (opus for 🔴 High risk)
```
Total testing time per feature: ~20 minutes

### Regulated Industry (Healthcare, Finance, etc.)
```
Personas: All 4 + domain expert persona
Tiers: All 3
Dimensions: All 6
Pass threshold: 25/30
Auto-fail on any accessibility issue: YES
Persona model: opus for all personas
Human approval required for: ALL features
```
Total testing time per feature: ~35 minutes

### Professional / Trained-User App (e.g., PhotoSortVision)
```
Personas: None (Insider covers functional testing exhaustively)
Tiers: Tier 3 — Insider only (exhaustive element audit)
Dimensions: All 6
Pass threshold: 25/30
Auto-fail on any broken element: YES
Persona model: opus (Insider needs full reasoning for spec comparison)
```
Total testing time per feature: ~15-25 minutes
Why this works: users are trained professionals who know the workflow. The Insider tests every button, toggle, live preview, and keyboard shortcut against the spec. No need for Cold tier "can a stranger figure this out?" testing — your users aren't strangers. Catches all functional bugs that other tiers would also find, plus spec compliance bugs that only the Insider can find.

### MVP / Prototype (Just Ship It)
```
Personas: Jordan only
Tiers: None
Dimensions: Functional only
Pass threshold: 10/30
Model: sonnet
```
Total testing time per feature: ~3 minutes

---

*Test Pilot Configuration v0.1.0*
*Part of the Test Pilot Loop — Flight Deck by Huan Su.*
*Everything is adjustable. Start light, add rigor as your project matures.*
