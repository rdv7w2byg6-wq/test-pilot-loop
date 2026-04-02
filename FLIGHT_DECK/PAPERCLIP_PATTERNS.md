# 📎 Paperclip Patterns — Future Additions for Flight Deck v2

> **These are Paperclip-inspired patterns that can be adopted into Flight Deck individually, at any time.**
> Each pattern is self-contained. Add one, add all, or add none — they don't depend on each other.
> Inspired by [Paperclip](https://github.com/paperclipai/paperclip) orchestration platform.

---

## Pattern 1: Atomic Task Checkout

**Problem:** With multiple coding agents (Claude Code + Codex) watching FLIGHT_PLAN.md, both could start fixing the same bug simultaneously — wasting tokens and creating merge conflicts.

**Solution:** Add a `checked_out_by` field to each FINDING. Before starting work, the agent writes its name. If another agent already checked it out, skip to the next unchecked item.

**Add to FINDINGS format:**

```
[2] severity: major | parenthetical_name_not_stripped
    CH05: OCR truncated "(Lou" without closing paren...
    File: NameParser.swift
    goal: Mission > Photo Import > Name Display
    found_by: cowork | status: open
    checked_out_by: none
    checkout_time: none
```

**Rules:**
- Before fixing: write `checked_out_by: [agent_name]` and `checkout_time: [timestamp]`
- If already checked out by another agent → skip, find another unchecked item
- Stale checkout (older than 30 minutes with no progress) → can be reclaimed
- On completion: update `status: fixed` and `checked_out_by: none`

**Where it goes:** Update the FINDINGS section format in FLIGHT_PLAN.md.

---

## Pattern 2: Per-Task Cost Tracking

**Problem:** You know how much each agent spent total, but not which bugs are expensive. A single bug eating $8 across 4 fix attempts should be escalated, not retried again.

**Solution:** Track cumulative cost per finding. Set a per-finding cost threshold. Auto-escalate when exceeded.

**Add to FINDINGS format:**

```
[2] severity: major | parenthetical_name_not_stripped
    ...
    status: open | retest_failures: 2
    cost_to_date: $8.40
    fix_attempts: 4
    cost_threshold: $10.00
```

**Rules:**
- Each time an agent works on a finding, it adds the estimated token cost to `cost_to_date`
- If `cost_to_date` exceeds `cost_threshold` → auto-escalate to human
- Default `cost_threshold`: $10.00 per finding (configurable in PREFLIGHT_CHECK.md)
- Findings with 0 fix attempts but high cost → signal that the diagnosis itself is hard

**Where it goes:** Update the FINDINGS section format in FLIGHT_PLAN.md. Add default threshold to PREFLIGHT_CHECK.md.

---

## Pattern 3: Delegation to Specific Agents

**Problem:** Claude Code handles all fixes, but some tasks are better suited for Codex (e.g., OpenAI API work) or another specialist agent.

**Solution:** Add a `delegate_to` field. The brain (Claude Code) assigns specific findings to the best agent. Tagged agents pick up their items on their next heartbeat.

**Add to FINDINGS format:**

```
[2] severity: major | api_integration_broken
    ...
    delegate_to: codex
    status: open
    reason: "Involves OpenAI API — Codex has better context"
```

**Add to NEXT_ACTION:**

```
NEXT_ACTION_FOR_CLAUDE_CODE:
  "Fix items [1] and [3]. Item [2] delegated to Codex."

NEXT_ACTION_FOR_CODEX:
  "Fix item [2] — OpenAI API integration. See FINDINGS [2] for details."
```

**Rules:**
- Only the brain (Claude Code) can write `delegate_to` tags
- Default is no delegation — Claude Code fixes everything unless it explicitly delegates
- Delegated agent follows same checkout protocol (Pattern 1)
- If delegated agent is not running → finding stays open, Claude Code picks it up after timeout

**Where it goes:** Update FINDINGS format and NEXT_ACTION section in FLIGHT_PLAN.md. Update second builder heartbeat in HEARTBEAT.md.

---

## Pattern 4: Session Persistence (SESSION_STATE.md)

**Problem:** Claude Code re-reads FLIGHT_PLAN.md cold on every heartbeat, wasting tokens re-parsing context it already knew.

**Solution:** Claude Code writes a session state file at the end of each heartbeat. On the next heartbeat, it reads SESSION_STATE.md first to restore context before reading FLIGHT_PLAN.md.

**New file: `FLIGHT_DECK/SESSION_STATE.md`**

```markdown
## Session State — Claude Code (Brain)
Last heartbeat: 2026-03-28 03:45 AM
Heartbeat count: 14

### Working Context
Currently fixing: FINDINGS [2] — parenthetical name regex
Progress: Identified root cause in NameParser.swift:51. Need regex that handles unclosed parens.
Files modified this session: NameParser.swift, NameParserTests.swift
Build status: Last build succeeded at 03:40

### What I Know (don't re-read)
- PRD fully loaded (PhotoSortVision, 17 chapters)
- GOAL_ANCESTRY loaded (3 objectives, 8 features)
- PREFLIGHT_CHECK loaded (Tier Gap target: 2.0)
- Personas: Pat (Cold) currently active

### What to Check (may have changed)
- FLIGHT_PLAN.md STATUS BLOCK (always check)
- FINDINGS (may have new entries from Cowork)
- BUDGET.md (may be approaching cap)
```

**Rules:**
- Claude Code writes SESSION_STATE.md at the end of every heartbeat
- On next heartbeat: read SESSION_STATE.md first, then read only what's flagged as "may have changed"
- If SESSION_STATE.md is missing or stale (older than 1 hour) → cold start, read everything
- Cowork has its own equivalent: TEST_STATE.md (already in v2 spec)

**Where it goes:** New file in FLIGHT_DECK/. Update Claude Code patrol protocol in FLIGHT_PLAN.md to read SESSION_STATE.md first.

---

## Pattern 5: Config Versioning in Flight Recorder

**Problem:** Flight Recorder snapshots code and build files, but not the testing configuration. If someone changes the pass threshold during an overnight run, tests could "pass" that shouldn't — and you can't tell the config changed.

**Solution:** Include Flight Deck config files in every Flight Recorder checkpoint.

**Updated checkpoint structure:**

```
checkpoint_NNN/
├── SNAPSHOT.md
├── timestamp.txt
├── stage.txt
├── git_commit.txt
├── files/                     ← build files (existing)
│   ├── PRD.md
│   ├── CLAUDE.md
│   └── STATE_MACHINE.md
├── config/                    ← NEW: Flight Deck config at this moment
│   ├── PREFLIGHT_CHECK.md     ← testing thresholds
│   ├── BUDGET.md              ← budget caps and actuals
│   ├── GATES.md               ← gate rules
│   └── GOAL_ANCESTRY.md       ← mission tree
├── test_state.md
└── budget_state.md
```

**Rules:**
- Every checkpoint includes a frozen copy of all Flight Deck config files
- On rollback, config files are restored along with build files
- SNAPSHOT.md notes any config changes since the previous checkpoint
- This prevents "config drift" — where testing rules change mid-run without anyone noticing

**Where it goes:** Update FLIGHT_RECORDER/README.md checkpoint structure. Update checkpoint creation logic in FLIGHT_PLAN.md patrol protocol.

---

## Pattern 6: Runtime Skill Injection Per Agent

**Problem:** Each agent type (Claude Code, Codex, Gemini, OpenClaw) needs manual setup instructions to participate in Flight Deck. Onboarding a new agent is a friction point.

**Solution:** Create per-agent skill files that auto-load when the agent starts. Drop the skill file into the agent's skills directory and it instantly knows the Flight Deck protocol.

**New folder: `FLIGHT_DECK/skills/`**

```
FLIGHT_DECK/skills/
├── claude/
│   └── flight-deck.md         ← auto-loads into ~/.claude/skills/
├── codex/
│   └── flight-deck.md         ← auto-loads into ~/.codex/skills/
├── gemini/
│   └── flight-deck.md         ← auto-loads into ~/.gemini/
├── cursor/
│   └── flight-deck.md         ← auto-loads into ~/.cursor/skills/
└── openclaw/
    └── flight-deck.md         ← already exists as OPENCLAW_SKILL.md
```

**Each skill file contains:**
- What Flight Deck is (one paragraph)
- This agent's role (brain, co-builder, test pilot, deliberator)
- The heartbeat protocol (specific to this agent type)
- How to read FLIGHT_PLAN.md
- How to write to FINDINGS and AGENT OUTPUT LOG
- Where to find the other Flight Deck files

**Symlink installation:**

```bash
# Claude Code
ln -s $(pwd)/FLIGHT_DECK/skills/claude/flight-deck.md ~/.claude/skills/flight-deck.md

# Codex
ln -s $(pwd)/FLIGHT_DECK/skills/codex/flight-deck.md ~/.codex/skills/flight-deck.md

# Or: add to PREFLIGHT_CHECK.md as a setup step
```

**Where it goes:** New `skills/` subfolder in FLIGHT_DECK/. Each file is a self-contained version of AUTOPILOT.md adapted for that specific agent type.

---

## Pattern 7: Portable Project Template (flight-deck init)

**Problem:** Adopting Flight Deck requires reading docs, copying files, understanding structure. High friction for new users.

**Solution:** A CLI command that scaffolds the entire Flight Deck structure into any project with one command.

**Usage:**

```bash
npx flight-deck init
```

**What it creates:**

```
FLIGHT_DECK/
├── FLIGHT_PLAN.md             ← with PROJECT_INFO blanks to fill
├── HEARTBEAT.md               ← unified protocol
├── AUTOPILOT.md               ← Claude Code loader
├── BUDGET.md                  ← default caps ($50 total)
├── GATES.md                   ← default gate rules
├── GOAL_ANCESTRY.md           ← template with Mission blank
├── PREFLIGHT_CHECK.md         ← default thresholds
├── TEST_FLIGHT_PROTOCOL.md    ← screen-by-screen protocol
├── TEST_STATE.md              ← empty, ready for Opus
├── FLIGHT_LOG.md              ← empty, append-only
├── FLIGHT_DEBRIEF.md          ← template
├── PILOT_HANDBOOK.md          ← persona testing guide
├── FLIGHT_PLANNING/
│   ├── README.md              ← deliberation protocol
│   ├── INBOX/                 ← empty, ready for drafts
│   ├── DISCUSSION/            ← empty, ready for debate
│   └── OUTPUT/                ← empty, ready for refined files
├── FLIGHT_RECORDER/
│   ├── README.md              ← Time Machine instructions
│   └── TIMELINE.md            ← empty timeline
├── skills/
│   ├── claude/flight-deck.md
│   ├── codex/flight-deck.md
│   └── openclaw/flight-deck.md
PERSONAS/
├── alex_power_user.md
├── jordan_everyday_user.md
├── pat_struggling_user.md
└── sam_accessibility_user.md
```

**Post-init instructions:**

```
✅ Flight Deck initialized!

Next steps:
1. Edit FLIGHT_DECK/FLIGHT_PLAN.md → fill in PROJECT_INFO
2. Edit FLIGHT_DECK/GOAL_ANCESTRY.md → define your mission and objectives
3. Edit FLIGHT_DECK/BUDGET.md → set your spending caps
4. Add @FLIGHT_DECK/AUTOPILOT.md to your CLAUDE.md
5. Run: fswatch -o FLIGHT_DECK/FLIGHT_PLAN.md  (start heartbeat)
6. Start building!
```

**Implementation:** This would be a small Node.js package (or bash script) published to npm. Similar to how Paperclip uses `npx paperclipai` for setup.

**Where it goes:** Separate npm package or a setup script in the repo root.

---

## Summary: When to Adopt Each

| Pattern | Complexity | Dependencies | Good Time to Add |
|---------|-----------|-------------|-----------------|
| 1. Atomic task checkout | Low | None | When you add a second builder |
| 2. Per-task cost tracking | Low | None | Before first overnight run |
| 3. Delegation to agents | Low | Pattern 1 | When you add a second builder |
| 4. Session persistence | Medium | None | After a few overnight runs (optimize later) |
| 5. Config versioning | Medium | Flight Recorder | When Flight Recorder is battle-tested |
| 6. Skill injection | Medium | None | When onboarding new agent types |
| 7. Portable template | High | Everything stable | When publishing to GitHub for others |
