# 🕐 FLIGHT RECORDER — THIS IS A TIME MACHINE

## For AI Agents and Humans: Read This First

**This folder lets you REWIND THE ENTIRE PROJECT to any previous moment.**

Every checkpoint below is a frozen snapshot of the project at a specific point in time — all build files, the git commit, the test state, the budget. If something goes wrong, you can go back to when things were working and try again.

**Think of it like macOS Time Machine, but for your entire development project.**

---

## HOW TO REWIND (For AI Agents)

Create a file called `ROLLBACK.md` in this folder:

```markdown
target: checkpoint_005
reason: "Sort feature fix broke the upload flow. Reverting to last known-good state."
```

The lead agent (Claude Code) will:
1. Read the target checkpoint's frozen files
2. Restore all build files (PRD, CLAUDE.md, state machine, etc.)
3. Git checkout to the commit saved in that checkpoint
4. Resume work from the stage that was active at that checkpoint
5. Log the rollback in FLIGHT_LOG.md
6. Create a NEW checkpoint recording the rollback itself

**Rewinding does NOT delete anything.** All future checkpoints remain in the timeline. The Flight Recorder is append-only — history is never erased, only added to.

---

## HOW TO BROWSE (For Humans)

1. Open `TIMELINE.md` for a quick summary of every checkpoint
2. Open any `checkpoint_NNN/SNAPSHOT.md` for details about that moment
3. Compare files across checkpoints to see what changed

---

## RULES

- **NEVER edit or delete a checkpoint** — they are immutable
- **NEVER edit TIMELINE.md entries** — only append new ones
- **Only the lead agent or human director can trigger a rollback**
- **Every rollback creates a new checkpoint** (the rewind itself is recorded)
- **Checkpoints are created automatically** at stage transitions, before tests, and after passes

---

## CHECKPOINT STRUCTURE

Each checkpoint folder contains:

```
checkpoint_NNN_description/
├── SNAPSHOT.md          ← What happened and why this checkpoint exists
├── timestamp.txt        ← When this moment was captured (ISO 8601)
├── stage.txt            ← Active stage: planning | flight | test_flight
├── git_commit.txt       ← Exact git commit hash
├── files/               ← Frozen copies of all build files at this moment
│   ├── PRD.md
│   ├── CLAUDE.md
│   ├── STATE_MACHINE.md
│   └── ...
├── test_state.md        ← Test progress at this moment (if in testing)
└── budget_state.md      ← How much budget was consumed
```

---

## WHEN CHECKPOINTS ARE CREATED

| Event | Checkpoint Created? |
|-------|:------------------:|
| Flight Planning complete (all agents agree) | ✅ Yes |
| Before each test cycle begins | ✅ Yes |
| After a test cycle passes | ✅ Yes |
| Before a major code change | ✅ Yes |
| After a rollback is executed | ✅ Yes |
| Human director manually requests | ✅ Yes |
| During normal coding (every small change) | ❌ No — use git for that |

---

## EXAMPLE TIMELINE

| # | Time | What Happened | Safe to Rewind Here? |
|---|------|---------------|:-------------------:|
| 001 | 02:30 | Planning complete — 3 agents agreed | ✅ Clean starting point |
| 002 | 03:15 | Upload feature built, pre-test | ✅ Code compiles |
| 003 | 03:28 | Cold user test FAILED | ⚠️ Has known failures |
| 004 | 03:45 | Drag-and-drop fix applied, pre-test | ✅ Fix applied |
| 005 | 03:58 | Cold user test PASSED | ✅ ⭐ Known-good state |
| 006 | 04:20 | Sort feature built, pre-test | ✅ New feature added |
| 007 | 04:35 | Sort test FAILED + upload regression | ⚠️ Regression present |
| 008 | 04:50 | ROLLBACK to 005 | ✅ Back to known-good |

**⭐ = Recommended rewind target** (everything was passing at this point)
