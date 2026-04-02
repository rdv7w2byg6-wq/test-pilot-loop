# 💓 Heartbeat — Unified Agent Protocol

> **Every agent follows the same heartbeat. Same rules. Same discipline.**
> The implementation differs (fswatch vs timed poll), but the protocol is identical.

---

## What Is a Heartbeat?

A heartbeat is a scheduled wake-up cycle. The agent wakes, checks its assigned communication file, acts if there's work, and goes back to sleep.

This replaces the ad-hoc patrol system from v1 with a consistent, predictable protocol that applies to **every agent** — whether it's Claude Code, Codex, Gemini CLI, or Cowork Opus.

**No agent runs continuously.** Every agent sleeps between heartbeats. This controls cost, prevents runaway loops, and makes overnight operation safe.

---

## The Heartbeat Cycle

Every agent, every wake-up, same six steps:

```
┌─────────────────────────────────────────────────────┐
│                  HEARTBEAT CYCLE                     │
│                                                     │
│  1. WAKE ─── triggered by fswatch event or timer    │
│     │                                               │
│  2. READ ─── check assigned communication file      │
│     │                                               │
│  3. DECIDE ─ is there work for me?                  │
│     │        ├─ YES → do the work → write results   │
│     │        └─ NO  → skip to step 5                │
│     │                                               │
│  4. WRITE ── update status, log actions taken        │
│     │                                               │
│  5. BUDGET ─ log tokens consumed this cycle          │
│     │        ├─ Under 80%  → normal, continue       │
│     │        ├─ 80-99%     → log WARNING, continue  │
│     │        └─ 100%       → STOP, write exit summary│
│     │                                               │
│  6. SLEEP ── wait for next trigger                   │
│     │                                               │
│  └──────────── repeat ──────────────────────────────┘
```

---

## Agent Heartbeat Configuration

### Coding Agents (Claude Code, Codex, Gemini CLI)

**Trigger:** `fswatch` — event-driven, zero cost while idle, instant response.

```bash
# Claude Code heartbeat
fswatch -o FLIGHT_DECK/FLIGHT_PLAN.md | while read; do
  claude "HEARTBEAT: Read FLIGHT_PLAN.md STATUS BLOCK and FEEDBACK QUEUE.
  Execute heartbeat protocol per HEARTBEAT.md.
  Log actions to FLIGHT_LOG.md."
done
```

```bash
# Codex heartbeat (for Flight Planning stage)
fswatch -o FLIGHT_DECK/FLIGHT_PLANNING/DISCUSSION/ | while read; do
  codex "HEARTBEAT: Read all files in FLIGHT_PLANNING/DISCUSSION/.
  Execute deliberation protocol per FLIGHT_PLANNING/README.md.
  Log actions to FLIGHT_LOG.md."
done
```

**Why fswatch:** CLI agents can use filesystem events natively. They wake only when something actually changes. Zero tokens burned while waiting.

**Install:** `brew install fswatch` (macOS) or `apt install fswatch` (Linux).

### Cowork Opus (Test Pilot)

**Trigger:** Timed poll — every 10 minutes, reads FLIGHT_PLAN.md.

Cowork cannot use fswatch (it runs as a desktop app with computer use, not a CLI process). It polls on a timer instead. Same protocol, different trigger mechanism.

**Standing instruction given to Cowork at session start:**

```
You are the Test Pilot for this project. You follow the heartbeat protocol.

HEARTBEAT CYCLE (every 10 minutes):
1. WAKE — your timer fired.
2. READ — open FLIGHT_PLAN.md in the project folder. Read the STATUS BLOCK.
3. DECIDE:
   - STATUS is READY_FOR_TEST or READY_FOR_RETEST → enter TEST MODE
   - STATUS is anything else → go back to sleep (do nothing for 10 minutes)
4. TEST MODE (when triggered):
   a. Read what needs testing (feature, app location, goal ancestry)
   b. Read TEST_STATE.md to restore context from previous cycles
   c. Find the running app on screen
   d. Run persona/tier tests per TEST_FLIGHT_PROTOCOL.md
   e. Write structured results to FLIGHT_PLAN.md FEEDBACK QUEUE
   f. Update TEST_STATE.md with current progress
   g. Update STATUS → TESTING_COMPLETE
5. BUDGET — note approximate token usage in BUDGET.md
6. SLEEP — wait 10 minutes, then repeat from step 1.

STOP CONDITIONS:
- GLOBAL_STOP: FROZEN → stop immediately, write frozen state to log
- Budget at 100% → stop, write exit summary
- STATUS is PASS or ALL_BUGS_VERIFIED → write TEST FLIGHT COMPLETE, stop patrol

Keep cycling until a stop condition is met or I tell you to stop.
This session may run overnight — pace yourself.
```

### Second Builder (Optional — Codex, another Claude Code, etc.)

**Trigger:** `fswatch` on FLIGHT_PLAN.md (same as primary Claude Code).

A second coding agent can watch the same file. To prevent conflicts:

- The lead builder (Claude Code) owns the STATUS BLOCK — only it writes status transitions
- The second builder reads the FEEDBACK QUEUE for tasks tagged to it
- If both agents see work, the lead builder's tag takes priority
- The second builder writes to AGENT OUTPUT LOG with its own agent name

```bash
# Second builder heartbeat
fswatch -o FLIGHT_DECK/FLIGHT_PLAN.md | while read; do
  codex "HEARTBEAT: Read FLIGHT_PLAN.md.
  Look for tasks tagged [CODEX] or [SECOND_BUILDER] in FEEDBACK QUEUE.
  If found: execute the task, write results to AGENT OUTPUT LOG.
  If not found: do nothing.
  Do NOT modify the STATUS BLOCK — that belongs to the lead builder."
done
```

---

## Heartbeat During Each Stage

### Stage 1: Flight Planning (Multi-LLM Deliberation)

All deliberation agents watch `FLIGHT_PLANNING/DISCUSSION/`:

| Agent | Watches | Trigger | Action on Wake |
|-------|---------|---------|----------------|
| Claude Code | `DISCUSSION/` | fswatch | Read all discussion files, write next round response |
| Codex | `DISCUSSION/` | fswatch | Read all discussion files, write next round response |
| Gemini CLI | `DISCUSSION/` | fswatch | Read all discussion files, write next round response |

**Turn-taking:** Before writing, agent creates `.thinking_agentname` lock file. Other agents wait if any `.thinking_*` file exists. Agent removes lock after writing. Stale locks (older than 5 minutes) are ignored.

**Agreement:** Each agent appends to `AGREEMENT.md` when satisfied. When all agents have appended AGREE, Stage 1 is complete.

**Budget:** Each agent tracks its deliberation spend. If an agent hits its Stage 1 budget cap, it writes a final position summary and exits deliberation. Remaining agents continue.

### Stage 2: Flight (Building)

Standard build-test loop — same as v1 but with Claude Code as lead brain:

| Agent | Watches | Trigger | Action on Wake |
|-------|---------|---------|----------------|
| Claude Code (lead) | `FLIGHT_PLAN.md` | fswatch | Read feedback, fix bugs, build, signal READY_FOR_TEST |
| Codex (second builder) | `FLIGHT_PLAN.md` | fswatch | Read tasks tagged [CODEX], execute, log results |

### Stage 3: Test Flight

Cowork Opus is the only agent active:

| Agent | Watches | Trigger | Action on Wake |
|-------|---------|---------|----------------|
| Cowork Opus | `FLIGHT_PLAN.md` | 10-min poll | Check STATUS, test if READY_FOR_TEST, write results |

Claude Code stays in heartbeat during Stage 3 — it reads Opus's test results and prepares fixes, but does NOT modify the STATUS BLOCK until Opus writes TESTING_COMPLETE.

---

## Heartbeat Confirmation (Required)

**No agent is assumed running until it confirms in writing.**

After heartbeat setup, each agent must write to `FLIGHT_PLAN.md` AGENT OUTPUT LOG:

```markdown
[timestamp] — [Agent Name]
Heartbeat configured.
Trigger: [fswatch on FLIGHT_PLAN.md | fswatch on DISCUSSION/ | 10-min timer]
Role: [Lead Builder | Second Builder | Test Pilot | Deliberation]
Budget cap: [$X.XX for this stage]
Standing by.
```

**Do not proceed to any stage until all required agents have confirmed.** The heartbeat system requires all agents actively listening. A single missing heartbeat breaks the feedback cycle.

---

## Heartbeat Escalation Triggers

These triggers apply to ALL agents, every heartbeat cycle:

| Condition | Action |
|-----------|--------|
| `GLOBAL_STOP: FROZEN` | Halt immediately, write frozen state to FLIGHT_LOG.md |
| Agent's stage budget at 80% | Log warning to FLIGHT_LOG.md, continue working |
| Agent's stage budget at 100% | Stop, write exit summary to FLIGHT_PLAN.md |
| No file changes for 30+ minutes during active stage | Write staleness warning to FLIGHT_PLAN.md |
| Build fails 2x on same fix | Escalate to human before third attempt |
| Bug fails retest 3x | Escalate to human — likely architectural issue |
| S1 Blocker found during testing | Alert human + pause immediately |
| Confidence < 60% on any action | Alert human before proceeding |
| Flight Planning: 3+ rounds with no new agreement | Escalate — agents may be deadlocked |

---

## Heartbeat vs. V1 Patrol — What Changed

| V1 Patrol | V2 Heartbeat |
|-----------|-------------|
| Each agent had its own patrol implementation | Unified protocol — same 6 steps for everyone |
| No budget tracking per cycle | Budget logged every heartbeat |
| No formal stop conditions | Explicit stop conditions (budget, GLOBAL_STOP, completion) |
| Cowork was brain + tester | Cowork is pure tester — only acts on READY_FOR_TEST |
| No deliberation stage | Heartbeat covers Flight Planning agents too |
| No persistent state between cycles | TEST_STATE.md carries forward across Opus cycles |
| Claude Code decided everything in patrol prompt | Claude Code is lead brain, but follows same heartbeat discipline |
| No turn-taking for multiple agents | Lock files prevent simultaneous writes |
| No staleness detection | 30-minute staleness warning |
| No deadlock detection for multi-agent discussion | 3-round no-progress escalation |

---

## Quick Reference — Agent Setup Commands

### Start all heartbeats for a full Flight Deck session:

**Terminal 1 — Claude Code (lead builder):**
```bash
fswatch -o FLIGHT_DECK/FLIGHT_PLAN.md | while read; do
  claude "HEARTBEAT: Execute heartbeat protocol per FLIGHT_DECK/HEARTBEAT.md. Role: Lead Builder."
done
```

**Terminal 2 — Codex (second builder / deliberation):**
```bash
# During Flight Planning:
fswatch -o FLIGHT_DECK/FLIGHT_PLANNING/DISCUSSION/ | while read; do
  codex "HEARTBEAT: Execute deliberation heartbeat per FLIGHT_DECK/HEARTBEAT.md."
done

# During Flight (building):
fswatch -o FLIGHT_DECK/FLIGHT_PLAN.md | while read; do
  codex "HEARTBEAT: Execute builder heartbeat per FLIGHT_DECK/HEARTBEAT.md. Role: Second Builder. Only act on tasks tagged [CODEX]."
done
```

**Terminal 3 — Cowork Opus (test pilot):**
```
Give Cowork the standing instruction from the "Cowork Opus" section above.
Cowork polls every 10 minutes — no terminal command needed.
```

### Stop all heartbeats:

- Set `GLOBAL_STOP: FROZEN` in FLIGHT_PLAN.md — all agents halt on next heartbeat
- Or: Ctrl+C each terminal (kills fswatch watchers)
- Or: Tell Cowork "stop patrol" directly

### Verify all heartbeats running:

Check FLIGHT_PLAN.md AGENT OUTPUT LOG for confirmation entries from every required agent.

---

## Design Principles

**Why not continuous?** Continuous agents burn tokens doing nothing. Heartbeats only cost tokens when there's actual work. Overnight, this is the difference between $5 and $50.

**Why same protocol for different agents?** Consistency. When something goes wrong at 3 AM, the human director reads FLIGHT_LOG.md and sees the same structured entries from every agent. No guessing which agent uses which format.

**Why budget tracking every cycle?** Because the alternative is waking up to a $200 bill. Per-cycle tracking means budget warnings appear in the log early, and hard caps prevent runaway spend.

**Why lock files for turn-taking?** Because fswatch triggers all watching agents simultaneously. Without locks, Claude Code and Codex could both write to DISCUSSION/ at the same instant, creating file conflicts. The `.thinking_*` convention is simple, reliable, and self-cleaning (5-minute staleness timeout).

**Inspired by Paperclip's heartbeat system** — adapted for file-based, zero-infrastructure operation. No server, no database, no API. Just files and fswatch.
