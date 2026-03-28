# Flight Plan — PhotoSortVision

> This is a worked example showing what a populated FLIGHT_PLAN.md looks like during a real project.

---

## STATUS BLOCK

```
GLOBAL STOP: ✅ ACTIVE
LAST PATROL: 2026-03-27 17:40
CURRENT PHASE: Test Complete — Conditional Pass
```

---

## TASK QUEUE

| ID | Task | Risk | Assigned To | Status |
|----|------|------|-------------|--------|
| T001 | Import screen — folder picker, drag-and-drop, photo count | 🟢 Low | Claude Code | ✅ Complete |
| T002 | PSV Engine scan — pre-filter, text recognition, name card detection, grouping | 🟡 Medium | Claude Code | ✅ Complete |
| T003 | Review screen — patient list, photo grid, assign/exclude/transform actions | 🟡 Medium | Claude Code | ✅ Complete |
| T004 | Sort execution — copy to destination, verify, undo support | 🔴 High | Claude Code | ✅ Complete |
| T005 | Sort report — summary stats, patient breakdown, PDF export | 🟢 Low | Claude Code | ✅ Complete |
| T006 | Fix BUG-001: Sort Confirmed double-click | 🟢 Low | Claude Code | ⏳ Awaiting Fix |
| T007 | Fix BUG-002: Placeholder text in patient list | 🟢 Low | Claude Code | ⏳ Awaiting Fix |
| T008 | Fix BUG-003: Status column truncation | ⚪ Trivial | Claude Code | ⏳ Awaiting Fix |

---

## AGENT OUTPUT LOG

### T001 — Import Screen

```
[2026-03-27 16:00] — Claude Code
Task ID: T001
Risk Level: 🟢 Low

Plan:
  • Build import screen with drag-and-drop zone and folder picker
  • Show photo count after folder selection
  • Add "Scan Now" CTA button

Output:
  Import screen complete. Drag-and-drop working, folder picker uses
  NSOpenPanel. Photo count updates live. "Scan Now" button triggers
  scan pipeline.

Files Modified:
  • PhotoSortVision/Views/ImportView.swift
  • PhotoSortVision/Models/PhotoImporter.swift

Tests: Passed — unit tests for file counting, extension filtering

Confidence: 90%
Status: 🟢 Auto-Proceeding to T002
```

### T002 — PSV Engine Scan

```
[2026-03-27 16:30] — Claude Code
Task ID: T002
Risk Level: 🟡 Medium

Plan:
  • Build scan pipeline: pre-filter → text recognition → name card detection → grouping
  • Show progress with sub-stage indicators
  • Handle "previously sorted" detection

Output:
  Scan pipeline complete. Four-stage progress shown in real time.
  Pre-filter checks brightness/blur. Text recognition uses Vision framework.
  Name card detection matches patient names. Grouping assigns photos to patients.
  "Previously sorted" detection checks destination folder manifest.

Files Modified:
  • PhotoSortVision/Models/PSVEngine.swift
  • PhotoSortVision/Models/TextRecognizer.swift
  • PhotoSortVision/Models/NameCardDetector.swift
  • PhotoSortVision/Views/ScanProgressView.swift

Tests: Passed — 12/12 unit tests, pipeline integration test with sample data

Confidence: 82%
  Unsure about edge case: what happens when brightness threshold is borderline.
  "Pre-filtered" stat label might be confusing (means "passed pre-filter" not
  "filtered out") — but this is a labeling concern, not a code issue.

Status: ⏳ Awaiting Review

Self-Assessment:
  The scan status line shows raw brightness values (e.g., "brightness=0.8").
  This is useful for debugging but might be too technical for end users.
  Flagging for reviewer to decide if this should be simplified.
```

### T003 — Review Screen

```
[2026-03-27 17:00] — Claude Code
Task ID: T003
Risk Level: 🟡 Medium

Plan:
  • Build master-detail layout: patient list (left), photo grid (right)
  • Add filter tabs: All, Assigned, Review, Excluded
  • Implement right-click context menu: Assign to, Exclude, Transform
  • Show status bar with counts

Output:
  Review screen complete. Patient list shows name, date, photo count,
  and assignment status. Photo grid loads thumbnails asynchronously.
  Context menu actions work. Filter tabs update both panels.

Files Modified:
  • PhotoSortVision/Views/ReviewView.swift
  • PhotoSortVision/Views/PatientListRow.swift
  • PhotoSortVision/Views/PhotoGridView.swift
  • PhotoSortVision/Models/ReviewFilter.swift

Tests: Passed — 8/8 tests for filtering, assignment, exclusion

Confidence: 75%
  The context menu is only accessible via right-click. No visible affordance
  for trackpad users who may not know about two-finger tap. Flagging this
  as a potential usability concern.

Status: ⏳ Awaiting Review

Self-Assessment:
  Patient list row shows "/description" and "/subfolder" as template text.
  These fields come from the name card detection pipeline and may not always
  be populated. Need to decide: hide empty fields, or show placeholder?
  Made a design decision to show placeholder — may be wrong.
```

### T004 — Sort Execution

```
[2026-03-27 17:15] — Claude Code
Task ID: T004
Risk Level: 🔴 High

Plan:
  • Build sort confirmation screen with summary and backup warning
  • Implement file copy with progress and verification
  • Add undo support (reversible for 30 minutes)

Output:
  Sort execution complete. Files copied to patient folders with
  per-file verification (checksum). Undo stores manifest for rollback.
  Confirmation screen shows backup warning and "Back to Review" option.

Files Modified:
  • PhotoSortVision/Views/SortConfirmationView.swift
  • PhotoSortVision/Views/SortProgressView.swift
  • PhotoSortVision/Models/FileSorter.swift
  • PhotoSortVision/Models/UndoManager.swift

Tests: Passed — 6/6 tests including undo rollback test

Confidence: 85%
  File copy and verification work correctly. Undo tested.
  One concern: the "Sort Confirmed" button on the Review screen
  transitions to this view — event handler attachment timing
  might be an issue if the view isn't fully loaded when the
  button becomes tappable.

Status: ⏳ Awaiting Review

Self-Assessment:
  Stats on completion screen show "8 Skipped" but don't break down
  whether these are excluded by user or duplicates. The Sort Report
  has the full breakdown. Completion screen could be more detailed.
```

---

## FEEDBACK QUEUE

### From Test Pilot → Claude Code (2026-03-27 17:40)

```
Test Result: ⚠️ CONDITIONAL PASS (24/30)

BUGS TO FIX (ordered by severity):

  1. BUG-001 [S3 🟡 Major] — "Sort Confirmed" requires double-click
     → FIX FIRST. First click produces no response. Second click works.
     → Likely event handler not attached on first render.
     → Root Cause: Event handler

  2. BUG-002 [S4 🔵 Minor] — "/description /subfolder" placeholder text visible
     → Fix after #1. Hide empty fields instead of showing template variables.
     → Root Cause: Missing validation

  3. BUG-003 [S4 🔵 Minor] — "Status" column header truncated to "St at us"
     → Column too narrow for header text. Widen or abbreviate.
     → Root Cause: Layout

  4. BUG-004 [S5 ⚪ Cosmetic] — "Avg: 0.0s / photo" rounding
     → Show 2 decimal places or omit if < 0.01s.
     → Root Cause: Missing validation

UX RECOMMENDATIONS (not bugs — for next iteration):
  • Disable "Sort New Only" when count is zero
  • Add subtitle to "92 To Review" explaining what it means
  • Add visible action affordance on photo thumbnails (not just right-click)
  • Simplify scan status line (hide brightness values)
  • Collapse PSV Engine settings under "Advanced" on first screen

HUMAN ESCALATION: None required for these bugs.
```

---

## How This Example Demonstrates the Framework

This FLIGHT_PLAN shows one complete cycle of the Test Pilot Loop:

1. **Tasks assigned** with risk levels (T001–T005)
2. **Builder output** with confidence scores and self-assessment
3. **Test Pilot evaluation** as Cold User through computer use
4. **Bug reports** with severity, root cause, and reproduction steps
5. **Feedback queue** directing the fix cycle
6. **Fix tasks created** (T006–T008) from test findings

The builder flagged two of the four bugs in self-assessment (placeholder text, event handler timing) but shipped anyway because tests passed. The Test Pilot caught them by actually using the app — exactly what the framework is designed to do.

---

*Part of the [Test Pilot Loop](https://github.com/suhuandds/test-pilot-loop) framework by Huan Su.*
