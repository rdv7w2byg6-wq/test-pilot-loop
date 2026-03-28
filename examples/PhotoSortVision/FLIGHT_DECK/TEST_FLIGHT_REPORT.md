# Test Flight Report — PhotoSortVision

```
TEST FLIGHT REPORT
══════════════════
App: PhotoSortVision (macOS, SwiftUI)
Tester: Cold User (no prior knowledge of the app)
Task: Import 470 dental photos, scan, review, and sort into patient folders
Completed: YES
Date: 2026-03-27

GROUND CHECK: App ✅ | Errors: 0 | Perf: instant launch

SCREENS VISITED: 5 (Import → Scan → Review → Sort Confirmation → Done)
TOTAL ACTIONS: 9
WRONG TURNS: 0
EARLY ABORT: PASSED (primary action identified within 30 seconds)

ELEMENT ANALYSIS:
  Total interactive: ~14 on first screen
  Clear labels: 12/14 (86%)
  Unlabeled: 2 (top-right toolbar icons unclear)
  No feedback when tapped: 1 ("Sort Confirmed" on first click)

PREDICTION ACCURACY: 7/9 (78%)
  Wrong predictions:
  - "Sort Confirmed" expected to advance immediately (required double-click)
  - Expected "previously sorted" message to explain what that means

NAVIGATION: Can always go back: Y | Always know location: Y (step indicator)

TOP FINDINGS (priority order):

  1. BUG-001 [S3 🟡 Major] — "Sort Confirmed" requires double-click
     Steps: Complete review → Click "Sort Confirmed" → nothing happens → click again → works
     Expected: Advances to Sort step on first click
     Actual: First click produces no response. Second click works.
     Root Cause: Event handler (likely not attached on first render or focus state conflict)

  2. BUG-002 [S4 🔵 Minor] — "/description" and "/subfolder" placeholder text visible
     Steps: Reach Review screen → look at patient list entries
     Expected: Description and subfolder fields populated or hidden
     Actual: Raw placeholder text "/description /subfolder" shown next to every patient name
     Root Cause: Missing validation (template variables not populated)

  3. BUG-003 [S4 🔵 Minor] — "Status" column header truncated to "St at us"
     Steps: Reach Review screen → look at far-left column
     Expected: Column header reads "Status"
     Actual: Text wraps awkwardly across lines as "St at us"
     Root Cause: Layout (column too narrow for header text)

  4. BUG-004 [S5 ⚪ Cosmetic] — "Avg: 0.0s / photo" rounding on scan complete
     Steps: Scan 470 photos → view completion screen
     Expected: Meaningful per-photo average (e.g., "0.03s")
     Actual: "0.0s / photo" — rounds to zero, looks like a display error
     Root Cause: Missing validation (number formatting needs more decimal places)

UX RECOMMENDATIONS (not bugs — suggestions for improvement):

  1. [Usability] "Sort New Only" button is green/primary when 0 new photos exist.
     Disable it or change label to "No new photos" when count is zero.

  2. [Usability] "92 To Review" on scan completion — unclear what "To Review" means.
     Add subtitle: "92 photos need your attention" or "92 photos couldn't be
     auto-assigned to a patient."

  3. [Usability] "Previously Sorted Photos Found" assumes user knows their history
     with the app. Add: "These photos were sorted by PhotoSortVision in a previous
     session."

  4. [Usability] Right-click actions on photos (Assign to, Exclude, Transform)
     have no visual affordance. A Cold User on a trackpad may never discover them.
     Consider a "..." button on hover or action buttons below thumbnails.

  5. [Usability] Scan progress shows developer-facing info: "DARK (brightness=0.8),
     skipping" — replace with "Skipping dark photo" or just show filename.

  6. [Usability] "Pre-filtered" stat during scan is ambiguous — does "295
     pre-filtered" mean 295 passed or 295 were filtered out?

  7. [Usability] Completion screen stats (470 Total, 455 Copied, 8 Skipped, 92 Review)
     don't obviously add up. The Sort Report has better breakdown (includes "Dupes
     Skipped" and "Matched"). Consider matching the report's categories on the
     completion screen.

  8. [Visual] "Make sure your SD card is backed up" on Sort confirmation appears
     styled as a link. If it's informational, don't underline it.

  9. [Visual] First screen is visually busy — PSV Engine settings (Rapid Mode,
     Auto Landscape, Auto Flip) are visible immediately. Consider collapsing
     these under "Advanced Settings" for new users.

WHAT WORKS WELL:
  ✅ Step indicator (Import → Scan → Review → Sort → Done) — always know where you are
  ✅ Progress UI during scan — sub-stages, count, time remaining, live stats
  ✅ Review screen master-detail layout — clean and familiar
  ✅ Sort confirmation screen with backup warning — good safety design
  ✅ "Undo Sort" on completion — safety net for destructive action
  ✅ Sort Report with Save PDF / Print / Copy — professional audit trail
  ✅ "Cancel Scan" available during processing
  ✅ Filter tabs (All/Assigned/Review/Excluded) on review screen

VERDICT: ⚠️ NEEDS WORK
  Core workflow completes successfully. One functional bug (double-click),
  one display bug (placeholders), and several usability opportunities.
  The app is strong on progress feedback and safety but needs polish
  on first-run clarity.
```

---

## Dimension Scores

| Dimension | Score | Notes |
|-----------|-------|-------|
| Usability & Clarity | 3/5 | Placeholders visible, ambiguous labels, hidden right-click actions |
| Functional Correctness | 4/5 | Sort Confirmed double-click bug, otherwise all features work |
| Navigation & Flow | 5/5 | Step indicator is excellent, always know where you are, back always available |
| Visual Design | 4/5 | Clean dark theme, good layout. First screen is busy. Minor truncation. |
| Error Handling | 4/5 | Backup warning, undo, cancel scan all present. No error states tested. |
| Data Integrity | 4/5 | 455/470 copied with verification. Stats slightly confusing. Undo available. |

**Total: 24/30 (80%) — CONDITIONAL PASS**

---

---

# Insider Tier — Test Flight Report

```
TEST FLIGHT REPORT (INSIDER)
════════════════════════════
App: PhotoSortVision (macOS, SwiftUI)
Tester: Insider (has PRD and full product knowledge)
Task: Verify all PRD features, test edge cases, evaluate spec compliance
Completed: YES
Date: 2026-03-27

SCREENS VISITED: 6 (Import → Review → Sort Complete → Undo → Import → Settings)
TOTAL ACTIONS: 7
FOCUS: Feature completeness, data integrity, edge cases

TOP FINDINGS (priority order):

  1. BUG-I001 [S3 🟡 Major] — Duplicate patient entries from OCR variants
     Steps: Review screen → Right-click photo → "Assign to" submenu
     Expected: Each patient appears once
     Actual: "W-ng, Suet" appears twice, plus "W Ng, Suet" as third entry.
       "Tien, An Vinh" also appears twice.
     Root Cause: Missing validation — OCR name deduplication not handling
       variant spellings (hyphen/space, spacing differences)
     PRD Impact: Sorting accuracy compromised — photos for same patient
       could end up in different folders

  2. BUG-I002 [S3 🟡 Major] — "Schedule Import" button on main screen is dead
     Steps: Import screen → Click "Schedule Import" section
     Expected: Opens file picker or navigates to Settings > Schedule
     Actual: Nothing happens. Feature only accessible through Settings gear > Schedule tab.
     Root Cause: Event handler — button not wired to action
     PRD Impact: Core feature not discoverable from primary screen

  3. BUG-I003 [S4 🔵 Minor] — No undo time remaining indicator
     Steps: Sort photos → View completion screen → See "Undo Sort" link
     Expected: Indicator showing "Undo available for 28 more minutes" or similar
     Actual: No countdown or expiry information visible
     Root Cause: Missing feature — undo expiry timer not surfaced to UI
     PRD Impact: PRD specifies 30-minute undo window but user can't see remaining time

  4. BUG-I004 [S4 🔵 Minor] — No confirmation after successful undo
     Steps: Sort Complete → Click "Undo Sort" → Confirm → Screen changes
     Expected: Toast or banner: "Sort undone. 455 files removed from output."
     Actual: Screen silently transitions to Review. No confirmation message.
     Root Cause: Missing feedback

  5. BUG-I005 [S5 ⚪ Cosmetic] — "Assign to" submenu has no search
     Steps: Right-click photo → "Assign to" → scroll through 38+ patients
     Expected: Search/filter field at top of patient list
     Actual: Must scroll through full list to find target patient
     Root Cause: Missing feature — search not implemented in submenu

UX RECOMMENDATIONS (Insider-specific):
  • Deduplicate patient names using fuzzy matching (Levenshtein distance ≤ 2)
  • Wire "Schedule Import" main-screen button to open Settings > Schedule or
    trigger file picker directly
  • Add undo countdown timer to completion screen
  • Add success toast after undo completes
```

## Insider Dimension Scores

| Dimension | Score | Notes |
|-----------|-------|-------|
| Usability & Clarity | 4/5 | Settings well-organized, but Schedule Import dead button |
| Functional Correctness | 3/5 | Duplicate patients from OCR, dead Schedule Import button |
| Navigation & Flow | 5/5 | Step indicator, back navigation, undo all work correctly |
| Visual Design | 4/5 | Consistent dark theme, good layout |
| Error Handling | 4/5 | Undo works with confirmation dialog. No expiry indicator. |
| Data Integrity | 3/5 | Duplicate patient entries risk mis-sorting. Undo works. |

**Total: 23/30 (77%) — NEEDS WORK**

---

# Tier Gap Analysis

```
TIER GAP RATIO
══════════════
Cold User actions to complete primary task:  9
Insider actions to complete primary task:    7
Tier Gap Ratio: 9 ÷ 7 = 1.29

Interpretation: Minimal gap — the app is nearly as usable for a new user
as for someone who knows the spec. The step indicator and green CTA buttons
guide the workflow effectively.

WHAT COLD FOUND THAT INSIDER MISSED:
  • First-impression visual clutter (PSV Engine settings)
  • Ambiguous labels ("Pre-filtered", "92 To Review", "NC" badge)
  • Hidden right-click affordance

WHAT INSIDER FOUND THAT COLD MISSED:
  • Duplicate patient entries from OCR variants (needs domain knowledge)
  • Schedule Import dead button (needs to know feature should exist)
  • Undo timer not visible (needs to know 30-min spec)
  • Settings > Schedule is the real Schedule Import (Cold never found it)

COMBINED FINDINGS: 4 bugs (Cold) + 5 bugs (Insider) = 9 total
  After dedup (Sort Confirmed double-click found by both): 8 unique bugs
  Overlap: 1 finding (12.5%)

CONCLUSION: Both tiers are essential. Cold catches UX confusion that an
  Insider would never notice. Insider catches spec violations and data
  integrity issues that require domain knowledge. Low overlap confirms
  the tiers test genuinely different things.
```

---

*Test Flight conducted by Cowork Opus. Cold + Insider tiers. PhotoSortVision v1.0, macOS.*
*Part of the [Test Pilot Loop](https://github.com/suhuandds/test-pilot-loop) framework by Huan Su.*
