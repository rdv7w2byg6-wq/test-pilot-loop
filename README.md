# 🧪 Test Pilot Loop

### AI that doesn't just write code — it flies the product first.

> *A framework where AI agents test software by using it — as different people, with different knowledge, measuring real UX.*

**Status: Experimental (v0.1.0) — Seeking real-world validation.**

> This is a docs-first framework: methodology, templates, and prompt assets — not a packaged software library. Copy the runtime files into your project repo to use the loop. See [What to Copy](#what-to-copy) below.

---

## The Problem

Your code agent writes beautiful code. Tests pass. CI is green. Then a real user opens the app:

> *"I don't know what this button does."*
> *"Where did my data go?"*
> *"How do I get back?"*

Code tests verify that functions return the right values. Nobody verifies that a human can actually use the result.

**The Test Pilot Loop fixes this.** After code is built, an AI uses the app through computer use — actually tapping, scrolling, typing, navigating — and gives honest human feedback. Not just bugs. Confusion, frustration, and friction.

**The app isn't done when code passes. It's done when the Test Pilot can use it.**

---

## How It Works

```
🛠️ Coding Agent builds feature
         ↓
✈️ Cowork (Test Pilot / Orchestrator)
         ↓
🧪 3-Tier Test Flight (Cold → Guided → Insider)
         ↓
📋 Flight Debrief (Unified + Prioritized)
         ↓
🛠️ Coding Agent implements fixes
         ↓
🔄 Verification Flight (Cowork retests)
         ↓
📈 Gap Check + Pass/Fail Decision
         ↓
🚀 Next cycle (if needed) — or ✅ Ship it
```

**The gap between the Cold user and the Insider is a quantitative measure of your UX quality.** We call it the **Tier Gap Ratio**: Cold user steps ÷ Insider steps. When it's 3x, your app depends on prior knowledge. When it's 1.5x, your app is genuinely intuitive.

---

## Three Levels of Use

### 🛫 Quick Flight — Zero Setup, 5 Minutes

Three Claude models (Opus, Sonnet, Haiku) each test your app independently. Each model's cognitive style naturally creates a different perspective:

- **Opus** overthinks — catches design flaws and subtle UX problems
- **Sonnet** follows the flow — catches what a normal user hits
- **Haiku** skims — catches what someone who gives up in 10 seconds would miss

No persona files. No configuration. Just three models, three honest reports.

**→ [Start here: QUICK_FLIGHT.md](QUICK_FLIGHT.md)**

---

### 🧪 Test Pilot Loop — Continuous Build-Test-Fix Cycle

Cowork Opus stays open as the brain. After each build, it test-drives the app three times — each time constrained to a different knowledge level. Writes structured feedback to a shared file. Code agent reads it, fixes, signals "ready for retest." Opus retests. Loop until it passes.

**One session. No switching. Continuous feedback loop.**

> **Canonical mode:** One Cowork Opus orchestrator drives the app through computer use and runs all evaluation passes sequentially. Persona subagent files (in `PERSONAS/`) are an optional helper pattern for code-side evaluation — the orchestrator is the default.

- **Personas** model **who** the tester is (expert, everyday, struggling, accessibility)
- **Knowledge tiers** model **what** the tester knows (nothing, user manual, full PRD)

The combination answers: can THIS person, with THIS much knowledge, use the app?

**→ [Main protocol: TEST_PILOT_LOOP.md](TEST_PILOT_LOOP.md)**

---

### 🏗️ Flight Deck — Full Framework

When you need maximum rigor: multi-agent orchestration, [Ralph Wiggum](https://github.com/anthropics/claude-code/blob/main/plugins/ralph-wiggum/README.md) build loop integration, kill switch, risk-tiered approval, persona subagents with deep behavioral rules, WCAG accessibility testing, and overnight autonomous operation.

| File | What It Is |
|------|-----------|
| [FLIGHT_PLAN.md](FLIGHT_DECK/FLIGHT_PLAN.md) | Full orchestration — Cowork brain, agents, risk tiers, kill switch, crash recovery |
| [PILOT_HANDBOOK.md](FLIGHT_DECK/PILOT_HANDBOOK.md) | Persona-based testing — 4 personas, 6 dimensions, scoring |
| [FLIGHT_DEBRIEF.md](FLIGHT_DECK/FLIGHT_DEBRIEF.md) | UX findings, bug classification, severity/priority, escalation outcomes |
| [FLIGHT_LOG.md](FLIGHT_DECK/FLIGHT_LOG.md) | Knowledge-tiered UX testing — 3 tiers, agent-steps, Tier Gap Ratio |
| [PREFLIGHT_CHECK.md](FLIGHT_DECK/PREFLIGHT_CHECK.md) | Configuration — choose personas, tiers, dimensions, thresholds |
| [AUTOPILOT.md](FLIGHT_DECK/AUTOPILOT.md) | Claude Code companion — teaches your code agent the protocol |
| [TEST_FLIGHT_PROTOCOL.md](FLIGHT_DECK/TEST_FLIGHT_PROTOCOL.md) | How the test pilot tests — screen-by-screen execution protocol |

**→ [Full framework: FLIGHT_DECK/](FLIGHT_DECK/)**

### 🔌 Integrations

OpenClaw integration available — drop `OPENCLAW_SKILL.md` into any OpenClaw agent system to run Test Pilot Loop flights without additional setup.

| File | What It Is |
|------|-----------|
| [OPENCLAW_SKILL.md](OPENCLAW_SKILL.md) | OpenClaw skill — self-contained version for OpenClaw agent systems |

### 📂 Worked Example

See `examples/PhotoSortVision/` for a complete worked example showing every framework artifact filled in with real test data — task queue, build output, test flight debrief, bug reports, and feedback cycle. Start with the [example README](examples/PhotoSortVision/README.md).

---

## What to Copy

This repo contains both **reference docs** (explaining the methodology) and **runtime files** (to copy into your project). Here's what goes where:

| Copy This | Into Your Project | Purpose |
|-----------|-------------------|---------|
| `CLAUDE.md` | Project root | Claude Code entrypoint (imports AUTOPILOT.md) |
| `FLIGHT_DECK/AUTOPILOT.md` | `FLIGHT_DECK/AUTOPILOT.md` | Builder agent instructions |
| `FLIGHT_DECK/FLIGHT_PLAN.md` | `FLIGHT_DECK/FLIGHT_PLAN.md` | Shared communication hub (edit per project) |
| `FLIGHT_DECK/PREFLIGHT_CHECK.md` | `FLIGHT_DECK/PREFLIGHT_CHECK.md` | Test configuration (edit per project) |
| `PERSONAS/agents/*.md` | `.claude/agents/` | Persona subagent files |
| `PRD.md` (you write this) | Project root | Your product requirements |

The remaining files (`TEST_PILOT_LOOP.md`, `QUICK_FLIGHT.md`, `PILOT_HANDBOOK.md`, `FLIGHT_DEBRIEF.md`, `FLIGHT_LOG.md`, `TEST_FLIGHT_PROTOCOL.md`, `OPENCLAW_SKILL.md`) are reference docs — read them to understand the framework, but they don't need to live in your project.

---

## Persona Subagents

Ready-to-use Claude Code subagent files. Drop into `.claude/agents/` and spawn them by name.

**Core 4 Personas:**

| File | Persona | Tests For |
|------|---------|-----------|
| [test-pilot-alex.md](PERSONAS/agents/test-pilot-alex.md) | 🟢 Power User | Shortcuts, batch ops, efficiency |
| [test-pilot-jordan.md](PERSONAS/agents/test-pilot-jordan.md) | 🟡 Everyday User | Confusing labels, broken happy path |
| [test-pilot-pat.md](PERSONAS/agents/test-pilot-pat.md) | 🔴 Struggling User | Cognitive overload, scary jargon |
| [test-pilot-sam.md](PERSONAS/agents/test-pilot-sam.md) | ♿ Accessibility User | WCAG compliance, keyboard traps |

**Knowledge Tiers:**

| File | Tier | Knowledge |
|------|------|-----------|
| [test-pilot-tier1-cold.md](PERSONAS/agents/test-pilot-tier1-cold.md) | Cold | Knows nothing |
| [test-pilot-tier2-guided.md](PERSONAS/agents/test-pilot-tier2-guided.md) | Guided | Has user manual |
| [test-pilot-tier3-insider.md](PERSONAS/agents/test-pilot-tier3-insider.md) | Insider | Has full PRD |

**Archetype Overlays:** [Creator / Admin / Consumer](PERSONAS/archetypes/archetype-overlays.md) — layer on top of any persona based on what feature is being tested.

**→ [Persona docs: PERSONAS/](PERSONAS/)**

---

## Requirements

This framework requires **Claude with computer use capability:**

- **Cowork (Claude Desktop app)** with computer use enabled — for the Test Pilot to see and interact with your app
- **Claude Code** — for building features and running subagents
- **A running instance of your app** visible on screen:

| Project Type | How to Launch for Testing |
|---|---|
| macOS app | Build and run in Xcode — app window visible on desktop |
| iOS app | Run in Xcode Simulator — simulator window visible on screen |
| Web app | Start dev server, open `localhost` in browser |
| Website | Open the URL in Safari or Chrome |
| Android app | Run in Android Studio emulator |
| CLI tool | Open Terminal, ready for commands |

**Key:** The app must be on the **same Mac** where Cowork is running and **visible on screen** (not minimized). Cowork interacts with what it can see.

> **Architecture note:** In this reference implementation, Cowork does not directly operate coding-agent terminals. Coordination happens through `FLIGHT_PLAN.md`, with the coding agent responsible for polling the file, acting on instructions, and reporting back. See [The Dual-Patrol Model](TEST_PILOT_LOOP.md#the-dual-patrol-model) for details.

The Test Pilot Loop is not a fully automated plug-and-play system (yet). It's a governed framework that requires a human director to launch the app, set up sessions, and make final calls. The AI handles the testing; you handle the setup.

> **Note:** The methodology is agent-agnostic. This repo is a reference implementation using Claude Code + Cowork. The concepts (persona-based testing, knowledge tiers, structured feedback loops) can be adapted to other AI agents with computer use capabilities.

---

## The Adoption Path

```
🛫 Quick Flight          "Let me try this in 5 minutes"
         ↓                     
🧪 Test Pilot Loop       "This works. I want continuous testing."
         ↓                     
📋 Personas               "I want deeper behavioral testing."
         ↓                     
🏗️ Flight Deck            "I need full governance for my team."
```

Start wherever makes sense for your project. Each layer is compatible with the ones above and below it.

---

## Repository Structure

```
test-pilot-loop/
├── README.md                              ← You are here
├── QUICK_FLIGHT.md                        ← 🛫 Zero-setup 3-model test
├── TEST_PILOT_LOOP.md                     ← 🧪 Core protocol (continuous loop)
├── CLAUDE.md                              ← Loader for Claude Code (imports AUTOPILOT.md)
├── FLIGHT_DECK/                           ← 🏗️ Full framework
│   ├── FLIGHT_PLAN.md                     ← Multi-agent orchestration + escalation rules
│   ├── PILOT_HANDBOOK.md                  ← Test personas, scoring, evaluation dimensions
│   ├── FLIGHT_DEBRIEF.md                  ← UX findings, bugs, escalation outcomes
│   ├── FLIGHT_LOG.md                      ← Cross-run metrics + Tier Gap tracking
│   ├── PREFLIGHT_CHECK.md                 ← Test configuration + run setup
│   ├── AUTOPILOT.md                          ← Claude Code operating instructions
│   └── TEST_FLIGHT_PROTOCOL.md               ← Screen-by-screen execution protocol
├── PERSONAS/                              ← 📋 Persona subagent files
│   ├── README.md                          ← Install guide + usage
│   ├── agents/                            ← Drop into .claude/agents/
│   │   ├── test-pilot-alex.md             ← 🟢 Power User
│   │   ├── test-pilot-jordan.md           ← 🟡 Everyday User
│   │   ├── test-pilot-pat.md              ← 🔴 Struggling User
│   │   ├── test-pilot-sam.md              ← ♿ Accessibility User
│   │   ├── test-pilot-tier1-cold.md       ← Tier 1 (knows nothing)
│   │   ├── test-pilot-tier2-guided.md     ← Tier 2 (has manual)
│   │   └── test-pilot-tier3-insider.md    ← Tier 3 (has PRD)
│   └── archetypes/
│       └── archetype-overlays.md          ← Creator / Admin / Consumer
├── OPENCLAW_SKILL.md                      ← OpenClaw skill version
├── examples/
│   └── PhotoSortVision/                   ← Worked example (real test data)
│       ├── README.md
│       ├── PRD.md
│       ├── CLAUDE.md
│       ├── FLIGHT_DECK/
│       │   ├── FLIGHT_PLAN.md             ← Populated: tasks, output, feedback
│       │   ├── TEST_FLIGHT_REPORT.md      ← Full debrief with dimension scores
│       │   └── PREFLIGHT_CHECK.md
│       └── PERSONAS/
│           └── jordan.md
├── CITATION.cff                           ← Citation metadata
└── LICENSE                                ← MIT
```

---

## What Makes This Different

| What exists today | What Test Pilot Loop adds |
|---|---|
| Unit tests check code correctness | **Test Pilot checks if a human can use it** |
| Linters check style | **Test Pilot checks if labels make sense** |
| CI validates that code compiles | **Test Pilot validates that users can succeed** |
| Code review checks implementation | **Test Pilot checks experience** |
| QA testers know the product | **Cold User knows nothing — tests real discoverability** |

This repo combines: live app testing through computer use + persona-based behavioral evaluation + knowledge-tiered UX measurement + structured feedback loops back to the coding agent.

---

## Origin

This framework was created by a developer who got tired of shipping features that passed every test but confused every user.

The core insight: **the person who builds the software is the worst person to test it.** They know too much. They can't see what's confusing because nothing confuses them.

So we built an AI test pilot that operates under a constrained context — no prior knowledge of the product, no insider assumptions — and evaluates the app through computer use the way a new user would encounter it.

---

## Safety and Privacy

This framework uses **computer use** — the AI can see your screen, click, and type. Keep these guidelines in mind:

- **Use synthetic or test data** when running test pilots. Don't expose real user data, credentials, or PII to the testing environment.
- **Run in a non-production environment.** Use simulators, dev servers, or staging builds — not production systems with live data.
- **Cowork is a research preview** (macOS only as of March 2026). Anthropic recommends dedicated low-privilege environments for computer use, human confirmation for consequential actions, and avoiding sensitive data on screen during sessions.
- **Review screenshots.** Test pilot sessions capture screenshots in `AUDIT_SCREENSHOTS/`. Review these before sharing or committing to ensure no sensitive data was captured.

---

## Contributing

This is experimental. We're looking for:

- **Real test reports** — run the framework, share your results
- **Domain persona packs** — healthcare, fintech, education, gaming
- **Archetype expansions** — new archetypes beyond Creator/Admin/Consumer
- **Integration guides** — how to use this with Cursor, Windsurf, and other coding agents
- **Bug reports** — inconsistencies, broken cross-references, unclear instructions

---

## License

MIT — use it, fork it, improve it, ship better software.

---

*Built with Claude Code, Cowork, and too many Xcode simulator crashes.*
*Test Pilot Loop by Huan Su · 2026*
