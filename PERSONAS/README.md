# 👥 Test Pilot Personas

Behavioral prompt packs for the Test Pilot Loop framework.

These are **Claude Code subagent files** — structured persona prompts that shape how each agent evaluates your app. They define the tester's mindset, behavioral rules, and report format.

> **Canonical mode:** The default testing method is one Cowork Opus orchestrator that drives the app through computer use and adopts each persona's behavioral rules in sequential passes. Persona subagent files are an **optional helper pattern** — they run in Claude Code using `tools: Read, Bash, Glob, Grep` for code-side evaluation and report generation, not for direct UI interaction. The orchestrator provides the eyes and hands; the personas provide the behavioral constraints.

## Quick Install

Copy the persona agents into your Claude Code project:

```bash
# Copy all persona agents
cp PERSONAS/agents/*.md your-project/.claude/agents/

# That's it. Claude Code will discover them automatically.
```

## What's Included

### Core 4 Personas (Use for Every Project)

| File | Persona | Tests For |
|------|---------|-----------|
| `test-pilot-alex.md` | 🟢 Power User — skips tutorials, wants shortcuts | Efficiency, batch ops, missing shortcuts |
| `test-pilot-jordan.md` | 🟡 Everyday User — follows the obvious path | Confusing labels, broken happy path, missing feedback |
| `test-pilot-pat.md` | 🔴 Struggling User — afraid to break things | Cognitive overload, scary jargon, missing undo |
| `test-pilot-sam.md` | ♿ Accessibility User — strict WCAG compliance | Keyboard traps, contrast, focus indicators, screen reader |

### Knowledge Tiers (Use for UX Phase Testing)

| File | Tier | Knowledge Level |
|------|------|-----------------|
| `test-pilot-tier1-cold.md` | 🔴 Cold User | Knows NOTHING — app name only |
| `test-pilot-tier2-guided.md` | 🟡 Guided User | Has user manual / README only |
| `test-pilot-tier3-insider.md` | 🟢 Insider | Has full PRD |

### Archetype Overlays (Layer on Top of Personas)

| File | Archetype | When to Apply |
|------|-----------|---------------|
| `archetype-overlays.md` | Creator / Admin / Consumer | Based on which feature is being tested |

## How to Use

### Functional Testing (Phase 1)
Run each persona sequentially (one at a time — each evaluates the app with different behavioral rules):
```
Test [feature] with each persona sequentially:
  Run 1: @test-pilot-alex — test as Power User → capture report → reset app
  Run 2: @test-pilot-jordan — test as Everyday User → capture report → reset app
  Run 3: @test-pilot-pat — test as Struggling User → capture report → reset app
  Run 4: @test-pilot-sam — test as Accessibility User → capture report
Compare all 4 reports.
```

### UX Testing (Phase 2)
Run each knowledge tier sequentially:
```
Test [feature] with each tier sequentially:
  Run 1: @test-pilot-tier1-cold — gets NO documentation → report → reset app
  Run 2: @test-pilot-tier2-guided — gets user manual → report → reset app
  Run 3: @test-pilot-tier3-insider — gets PRD.md → report
Compare all 3 reports. Calculate Tier Gap Ratio.
```

### Adding Archetype Overlay
Include the archetype instructions in the spawn prompt:
```
Use @test-pilot-pat to test the photo import feature.
Pat is acting as a CREATOR (putting data into the system).
Additional Creator behaviors: try empty submissions, wrong file
types, mid-task abandonment, special characters.
```

## Adding Domain Personas

Create your own persona file in `.claude/agents/`:

```markdown
---
name: test-pilot-[name]
description: [one line description]
model: sonnet
tools: Read, Bash, Glob, Grep
---

# You Are [Name] — [Role]

## Your Mindset
"[one sentence that captures how this person thinks]"

## Your Background
- [3-5 bullet points about who they are]

## STRICT BEHAVIORAL RULES
- [specific actions the AI MUST take]
- [specific things the AI MUST NOT do]
- [emotional reactions to specific triggers]

## After Testing, Report
[structured report template]
```

The more specific the behavioral rules, the more different the test results will be from other personas.

---

*Part of the Test Pilot Loop by Huan Su.*
