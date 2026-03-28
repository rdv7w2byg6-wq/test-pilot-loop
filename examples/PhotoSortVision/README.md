# PhotoSortVision — Worked Example

This is a complete worked example of the **Test Pilot Loop** framework applied to a real project.

PhotoSortVision is a macOS app that imports clinical dental photos, detects patient name cards via on-device vision, and sorts originals into patient-specific folders. This example uses real test data from an actual test flight — not fabricated artifacts.

## What This Example Shows

The full Test Pilot Loop cycle, start to finish:

```
PRD → Task Queue → Build → Test Flight → Debrief → Bug Reports → Fix Tasks
```

Every file in this example is filled in with real data from an actual test session showing exactly what each artifact looks like during a live project.

## File Map

| File | What It Shows |
|------|--------------|
| `PRD.md` | Product requirements — what gets built |
| `CLAUDE.md` | Project root config — imports Autopilot |
| `FLIGHT_DECK/FLIGHT_PLAN.md` | The full communication hub — task queue, output logs, feedback queue |
| `FLIGHT_DECK/TEST_FLIGHT_REPORT.md` | Complete test flight debrief — findings, dimension scores, verdict |
| `FLIGHT_DECK/PREFLIGHT_CHECK.md` | Testing depth settings for this project |
| `PERSONAS/jordan.md` | Everyday User persona used in testing |

## How to Read This Example

1. Start with `PRD.md` — understand what the app should do
2. Read `FLIGHT_DECK/FLIGHT_PLAN.md` from top to bottom — the living document. You'll see:
   - Tasks assigned to Claude Code with risk levels
   - Build output with confidence scores and self-assessment
   - Test Pilot findings with severity-ranked bug reports
   - Feedback queue directing the fix cycle
3. Read `FLIGHT_DECK/TEST_FLIGHT_REPORT.md` — the full test debrief with dimension scores
4. Notice how the framework caught real UX problems that unit tests would never find

## Key Takeaway

All unit tests passed. Zero compiler warnings. The Test Pilot still found 4 bugs and 9 UX recommendations — all of them problems that only surface when someone uses the app through computer use without prior knowledge. Two of the bugs were flagged by the builder in self-assessment but shipped anyway because code tests passed. The Test Pilot caught them by actually clicking through the app.

**Test result: Conditional Pass (24/30, 80%)**

---

*Part of the [Test Pilot Loop](https://github.com/suhuandds/test-pilot-loop) framework by Huan Su.*
