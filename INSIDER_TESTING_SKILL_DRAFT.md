# Insider Testing Skill — Test Pilot Loop
## Draft v0.4 — For Review Before Integration

---

## What Is Insider Testing?

Insider testing is not a checklist. It is a **thinking discipline**.

An Insider understands the product's purpose, uses it like someone who cares about the outcome, and evaluates whether the app actually does its job — not just whether buttons click.

| Tier | Reads Spec? | Tests What? | Thinks About? |
|------|------------|-------------|---------------|
| Cold | No | Can I figure this out? | First impressions, discoverability |
| Guided | Partially | Does it match instructions? | Flows, labeling, expected behavior |
| **Insider** | **Full spec** | **Does it achieve its purpose?** | **Outcomes, logic, design, edge cases** |

The Insider is the only tier that asks: **"The app ran successfully — but did it produce the right result?"**

---

## Phase 0: Mandatory Setup — Goal Discovery Q&A

### Where This Fits

This phase is part of the **existing mandatory setup** in AUTOPILOT.md. The current setup has two steps:

```
AUTOPILOT.md Step 1: Read the project spec (CLAUDE.md, PRD.md)
AUTOPILOT.md Step 2: Set up auto-patrol (fswatch or /loop)
──────────── NEW ────────────
Step 1.5: Goal Discovery Q&A (this skill)
```

**After Claude Code reads the spec but BEFORE patrol starts**, Claude Code conducts a 6-question Q&A with the human. Questions 1-5 are generated from the spec. Question 6 is always the same: "Why are you testing?" Claude Code is the right agent for this because it just read the full spec — it understands the project deeply enough to ask *smart* questions, not generic ones.

### Why Claude Code Asks (Not Cowork)

Claude Code is the brain. It reads CLAUDE.md (every screen, every button, every state transition) and PRD.md (the product requirements). From this, it already knows:
- What the app is supposed to do
- What output it produces
- What formats/edge cases exist in the spec

But the spec doesn't tell Claude Code:
- What "correct" looks like in the human's specific context
- Which test data is available and what it covers
- Which edge cases the human has actually seen fail
- What the human cares about most

The Q&A fills those gaps. Claude Code reads the spec, then asks 5 targeted questions **based on what it read**.

### The 6 Questions

Claude Code generates Q1-Q5 dynamically from the spec. They are NOT generic templates — they reference specific features, formats, and edge cases from CLAUDE.md/PRD.md. Q6 is always asked verbatim.

The questions follow this structure, but Q1-Q5 content adapts to each project:

#### Q1: Goal & Output Verification
> "I've read the spec. The app's goal is [X]. After a test run, where should the tester verify the result, and what does correct output look like?"

Claude Code fills in [X] from the spec. The human confirms or corrects, and adds specifics about how to verify.

**Example for PhotoSortVision:**
> "I've read the spec. The app sorts dental photos into patient folders using name card OCR. After a sort, should the tester open the destination folder in Finder to verify photos are in the correct patient folders? What does 'correct' look like for your practice?"

#### Q2: Failure Investigation
> "The spec describes [failure mechanism]. When the tester finds items in [failure location], how deep should they investigate?"

Claude Code identifies where failures go from the spec. The human defines investigation depth.

**Example for PhotoSortVision:**
> "The spec says unmatched photos go to `_Review/` folder grouped by time window. When the tester finds photos there, should they open each one and figure out WHY — missing name card, OCR failure, gap threshold? Or just verify the count?"

#### Q3: Test Data & Known Hard Cases
> "I see the spec handles [formats/features]. What test data is available, and which cases are most likely to break?"

Claude Code lists what the spec claims to support. The human says what's actually testable.

**Example for PhotoSortVision:**
> "The spec lists 5 name card formats (A-E), CJK names, and multi-word last names like 'Rubio Diaz'. I see TEST_INPUT/ with 470 photos. Any additional test datasets? Which name card formats does your data actually contain? Any formats that have given you trouble?"

#### Q4: Critical Settings
> "These settings affect the output: [list from spec]. Which ones should the tester change and re-verify?"

Claude Code identifies output-affecting settings from the spec. The human prioritizes.

**Example for PhotoSortVision:**
> "I see these settings affect sorting output: gap threshold (1h default), filename separator, folder name format, Rapid Mode, and Auto Landscape. Which should the tester change between runs to verify they actually affect the output files?"

#### Q5: Out of Scope
> "I noticed these features in the spec that may not be built yet: [list]. Anything else the tester should skip? Anything that looks like a bug but is intentional?"

Claude Code flags post-MVP or in-progress items. The human confirms and adds to the ignore list.

**Example for PhotoSortVision:**
> "I see Schedule Import, Resident Program, metadata embedding, and SHA-256 duplicate detection listed as post-MVP. Anything else the tester should skip? Any behaviors that might look wrong but are intentional?"

#### Q6: Why Are You Testing?
> "Last question — what's your main concern? Why do you want this tested right now?"

This is the most important question. It's asked last because by now the human is in the right headspace — they've been thinking about goals, failures, and edge cases. Their answer reveals what they're actually worried about, which sets the tester's priority.

**This question is always asked exactly as written. It is not generated from the spec.** The spec can't tell Claude Code what the human is anxious about.

**Example answers and how they change testing priority:**

| Human says... | Tester prioritizes... |
|---------------|----------------------|
| "I'm not confident the OCR catches all name card formats" | Name card detection accuracy — test every format, study every missed card |
| "I just rebuilt the sort pipeline, worried about data loss" | File integrity — count every photo, verify nothing is lost or corrupted |
| "The app is for my dental assistant, she's not technical" | UX clarity — every label, every error message, every empty state |
| "I'm shipping this next week" | Full workflow reliability — happy path must be bulletproof |
| "I added a new feature and want to make sure it didn't break anything" | Regression — test existing features, not just the new one |
| "I want to know if the test pilot loop framework works" | Meta-testing — evaluate the framework itself alongside the app |

The answer gets saved as `### Testing Priority` in INSIDER_GOALS.md and is the FIRST thing the tester reads. It overrides default time allocation across phases.

### How Claude Code Generates Questions 1-5

After reading CLAUDE.md / PRD.md, Claude Code:

1. **Identifies the primary goal** — what is the app's core job? (Sort photos, generate reports, process data...)
2. **Identifies the output** — where does the result go? (Folder, database, file, API...)
3. **Identifies the failure path** — where do rejected/failed items go? (Review folder, error log, skip list...)
4. **Lists output-affecting settings** — which settings change what the output looks like?
5. **Lists spec features by status** — what's built, what's post-MVP, what's in progress?

Then it formulates Q1-Q5 using this knowledge. The questions demonstrate understanding and ask for the human's specific context. Q6 is always asked verbatim — it doesn't need spec knowledge, it needs the human's honest answer.

### Saving the Goals

Claude Code compiles the answers into `FLIGHT_DECK/INSIDER_GOALS.md`:

```markdown
# Insider Testing Goals — [Project Name]
## Generated: [date]
## Source: Goal Discovery Q&A (Claude Code + Human Director)

### ⚡ Testing Priority — READ THIS FIRST
[Why the human wants testing — from Q6]
[How this shifts time allocation and focus]

### Final Goal
[What success looks like — from Q1]

### Output Verification Method
[Where to check, what correct looks like — from Q1]
Verification location: [Finder path / URL / tool]
Verification method: [open folder / query database / compare files]

### Failure Investigation Protocol
[Where failures go, how deep to investigate — from Q2]
Failure location: [path]
Investigation depth: [count only / classify each / root-cause each]

### Test Data Inventory
[Available datasets and known hard cases — from Q3]
| Dataset | Path | Photos | Coverage | Notes |
|---------|------|--------|----------|-------|
| ... | ... | ... | ... | ... |

### Critical Settings to Verify
[Settings that affect output — from Q4]
| Setting | Default | Test Values | How to Verify |
|---------|---------|-------------|---------------|
| ... | ... | ... | ... |

### Out of Scope
[What to skip — from Q5]
- [feature 1] — reason
- [feature 2] — reason

### Goal Verification Checklist
[Auto-generated from all answers — the specific checks to run after EVERY test flight]
□ [check 1]
□ [check 2]
□ [check 3]
...
```

**This file is read by Cowork (the test pilot) before every test flight.** It grounds the tester in what matters for *this specific project*, as told by the human who built it.

### When to Re-Run the Q&A

The Q&A runs once during initial setup. Re-run it when:
- New test data is added (human says "I added TEST_INPUT 2")
- Major features are built that change the output
- Human changes their testing priorities
- First Q&A was too early (app wasn't built enough to answer meaningfully)

Append to INSIDER_GOALS.md rather than overwriting — track how goals evolve.

---

## Oracles — How the Tester Knows Something Is Wrong

An **oracle** is anything the tester compares against to decide if a result is correct or incorrect. Without oracles, the tester can only say "it didn't crash" — not "it did its job."

Insider testers use multiple oracles simultaneously:

| Oracle | What It Is | Example |
|--------|-----------|---------|
| **The Spec** | What CLAUDE.md / PRD.md says should happen | "Spec says Format B: 'Patient' on own line, name on next line" |
| **The Output** | The actual files, folders, data produced | Open Finder → check patient folders → are right photos inside? |
| **The Input** | What went in — compare against what came out | 470 photos in → 430 sorted + 35 review + 5 excluded = 470? |
| **Consistency** | Same action should produce same result | Sort twice with same data → same output? |
| **Comparable Product** | How similar apps handle the same case | "Every other photo app handles this name format" |
| **User Expectations** | What a reasonable user would expect | "I'd expect my photos to be there after sorting" |
| **History** | What happened in previous test flights | "This worked last flight — regression?" |

**The most powerful oracle for goal verification is the output itself.** The tester opens Finder, looks at what the app produced, and compares it against what *should* be there. This is not subjective — it's verifiable.

### When Oracles Conflict

Sometimes the spec says one thing but the output suggests another approach is better. Or the user expects behavior the spec doesn't define. When oracles conflict, the tester notes it and asks the human in Phase 8 (Post-Flight Q&A). Don't silently resolve conflicts — surface them.

---

## The Adaptive Pivot — When to Go Deep

From exploratory testing methodology: when a tester finds something unexpected, they don't just log it and move on. They **pivot** — immediately exploring related areas to understand the scope.

**The rule: One failure → test 5 related cases before moving on.**

| Tester finds... | Pivot to... |
|-----------------|-------------|
| One name card not detected | Test 5 more cards of the same format. Is it this card or this format? |
| One photo in wrong folder | Check 5 neighboring photos. Is it one mistake or a pattern? |
| Separator not updating in preview | Test date format, name order, tokens. Which settings update and which don't? |
| Slow performance on one dataset | Test same dataset size with different content. Is it size or content? |

**Why this matters for AI testers specifically:** AI agents tend to log an issue and proceed to the next checklist item. Human experts dig in. The pivot rule forces investigation before moving on. A single bug that reveals a pattern is worth more than 10 surface-level observations.

---

## The Five Lenses

Every screen, every feature, every output must pass through all five lenses. The Insider holds all five in mind simultaneously.

### Lens 1: Does It Work?
> "I clicked it. Did something happen?"

The baseline. Verify every interactive element responds. But this is the *floor*, not the ceiling.

- Buttons click and trigger the expected action
- Toggles change state and persist
- Inputs accept text and validate
- Navigation moves to the correct screen
- Progress indicators advance and complete

**This lens alone is NOT Insider testing. It's QA 101.**

### Lens 2: Does It Achieve the Goal?
> "The app said 'Done.' But did it actually do its job?"

**This is the most important lens. It is the reason Insider testing exists.**

The app exists to produce a result. After it finishes, leave the app and go verify the result independently. The UI saying "Sort Complete — 37 patients" is a claim. The Insider verifies the claim.

**Verify the output:**
- Open the output location (Finder, database, API, filesystem)
- Spot-check that results are correct, not just present
- Compare output against what you know should be there

**Investigate the failures:**
- Open the failure/review/error location
- For EACH failure item, ask: WHY is this here?
- Classify each failure: correct behavior? Engine limitation? Bug?
- Look for patterns: are all failures the same type? Same root cause?

**Reconcile the numbers:**
- Input count = output count + failure count + excluded count
- Any missing? That's a potential data loss bug — highest severity.

**Check the details:**
- Do filenames match the naming settings?
- Do folder names match expected patterns?
- Are duplicates handled correctly?
- Did transforms (rotation, flip) apply?

#### Example: Photo Sorter Goal Verification
```
GOAL: 470 photos sorted into correct patient folders

Step 1: Open destination folder in Finder
  □ Patient folders exist with correct names
  □ Open 3-4 folders, verify photos belong to that patient
  □ Check a name card photo — is it in the right folder?
  □ Check the LAST photo before a gap — is it assigned correctly?
  □ Check filenames: do they match the naming template?

Step 2: Open _Review folder in Finder
  □ How many photos? Does this match the app's count?
  □ Open each review subfolder (time-grouped)
  □ For each photo, determine WHY it's here:
    - No preceding name card? → correct behavior
    - Name card present but OCR failed? → engine issue (investigate)
    - Time gap exceeded threshold? → check if threshold is appropriate
    - Photo of a name card itself not detected? → detection issue
  □ Are there photos here that SHOULD have been sorted? → BUG

Step 3: Reconcile
  □ Sorted + Review + Excluded = 470? (input total)
  □ Any missing photos? → CRITICAL BUG (data loss)

Step 4: Study failed name cards
  □ If a name card went to review instead of triggering a patient:
    - Open the photo — what does the name card look like?
    - Is it a format the spec says should be detected (A/B/C/D/E)?
    - Is the text readable to a human? If yes but OCR missed it → engine bug
    - Is it a new format not in the spec? → note for spec update
    - Take a screenshot for the bug report
```

### Lens 3: Is It Hard to Use?
> "I got it done, but should it have been that difficult?"

Friction that doesn't cause failure but wastes time or causes confusion:

- Had to scroll excessively to see related information
- Had to click multiple times where one should suffice
- Labels that are technically accurate but not intuitive
- Information the user needs is hidden behind extra steps
- No visual feedback after an action (silence = confusion)
- Settings and their preview not visible at the same time
- Error messages that describe the error but not the fix

**Report UX friction even when the feature technically works.**

### Lens 4: Does It Make Sense?
> "This works, but should it exist? Does it contradict something else?"

Requires understanding the product holistically:

- **Redundant controls** — Two UI elements that do the same thing. Which wins if they conflict?
- **Contradictory settings** — Setting A implies X, Setting B implies Y. What happens with both active?
- **Orphaned features** — A setting exists but nothing uses it
- **Missing connections** — Feature A produces output that Feature B should consume, but doesn't
- **Naming inconsistencies** — Same concept, different words in different places
- **Broken mental models** — UI suggests one workflow, app works differently

**Ask: "If I were explaining this to a new user, would I have to say 'ignore that' or 'that's confusing but...'?"**

### Lens 5: What Else?
> "What hasn't been tested that should be?"

The creative lens:

- **Different datasets** — Does it work with dataset 2? Different formats, different edge cases?
- **Scale** — 10 items vs 1000. Does behavior change?
- **Sequence** — Run the same operation twice. Duplicates handled?
- **Recovery** — Interrupt mid-operation. What happens?
- **Empty states** — Zero input. Does it handle nothing gracefully?
- **Adversarial inputs** — Special characters, extreme lengths, missing metadata

---

## Insider Testing Protocol

### Phase 1: Goal Verification — Primary Dataset
**This is the most important phase. Do this first.**

Run the app's complete workflow end-to-end with the primary test dataset. Then:

1. **Leave the app.** Open Finder (or whatever shows the output).
2. **Verify the output** against the Goal Verification Checklist from INSIDER_GOALS.md.
3. **Investigate every failure** — not just count them, understand each one.
4. **Reconcile numbers** — input = output + review + excluded. Any missing = critical.
5. **Document findings** as OUTCOME entries (new category, see Reporting Format).

```
TIME ALLOCATION: Spend 40% of total testing time on this phase.
This is where the highest-value findings come from.
```

### Phase 2: Failure Deep-Dive
**Do not skip this phase.** Most testers verify success. Insiders investigate failure.

For each item in the failure/review bucket:

| Photo | Why Here? | Correct? | Root Cause | Actionable? |
|-------|-----------|----------|------------|-------------|
| DSC_0047 | No name card before it, 62 min gap | Yes | Gap threshold working | Could improve: smart suggestion |
| DSC_0102 | Name card present but not detected | **No — bug** | OCR missed "Patient" label | BUG: Format B not recognized |
| DSC_0200 | First photo in batch, no preceding context | Yes | Expected for first photo | Could improve: schedule matching |

For name cards that failed detection:
1. Open the photo — what does it look like?
2. Which format is it? (A/B/C/D/E per spec)
3. Can a human read it? If yes → engine should have caught it
4. What text would OCR produce? Is the parsing logic handling it?
5. Screenshot for the bug report

### Phase 3: Goal Verification — Second Dataset
Run the full workflow again with a **different** test dataset.

Purpose: surface data-dependent issues that Dataset 1 doesn't trigger.

After completion, repeat Phase 1 and Phase 2 against the second dataset's output. Then compare:

```
□ Same patients detected in both? Any discrepancies?
□ Any name card formats that work in one dataset but not the other?
□ Different camera EXIF — any timestamp issues?
□ Different file types — HEIC vs JPEG vs RAW handling?
□ Performance: faster or slower? Accuracy better or worse?
□ Any new failure patterns not seen in Dataset 1?
```

### Phase 4: Settings Impact Verification
Change key settings (from INSIDER_GOALS.md > Critical Settings) and verify they affect the output correctly.

For each critical setting:
1. Change the setting
2. Run the workflow (or relevant portion)
3. Verify the output reflects the setting change
4. Check the Live Preview if one exists — does it match reality?

```
Example for Photo Sorter:
□ Change gap threshold from 1h to 30min → re-scan → more photos in review? Correct?
□ Change filename separator → sort → check actual filenames in Finder
□ Change folder name format → sort → check actual folder names in Finder
□ Toggle Rapid Mode → re-scan → different detection results?
□ Change name order → sort → filenames reflect new order?
```

**Key principle: the test is in the output, not the UI.** If the Live Preview says one thing but the actual sorted files say another — that's the real bug.

### Phase 5: Screen-by-Screen Element Audit
Now do the detailed element verification. But augmented with all 5 lenses:

For each element, evaluate:

| Element | Works? | Affects output? | Easy to use? | Makes sense? | Notes |
|---------|--------|----------------|--------------|-------------|-------|
| Gap slider | ✓ | Yes — changes review count | ✓ | ✓ | |
| Name Order radio | ✓ | Redundant with token chips | ✓ | ✗ — redundant | NOTE-007 |
| Live Preview | ✓ partial | Shows filename but separator doesn't update | Hard — must scroll far | ✓ concept | BUG-013 |

For settings and controls, always ask:
- What does changing this actually DO to the final output?
- Can I verify the effect by checking the output folder?
- Does this duplicate another control?
- Would a user understand this without documentation?

### Phase 6: Edge Cases & Stress
Targeted tests for boundary conditions:

- Re-run with same data (duplicate handling — check output folder)
- Run with empty input folder
- Run with only name cards, no patient photos
- Run with only patient photos, no name cards
- Change settings between scan and sort — does output match which settings?
- Photos with no EXIF data
- Special characters in patient names — check folder names in Finder
- Very long patient names — check folder name truncation

### Phase 7: Report
Compile findings organized by **lens**, not just by screen.

### Phase 8: Evolve — Post-Flight Q&A With Human

**After delivering the report, the tester asks the human what to test next.**

This is not optional. The tester just spent hours with the app — they have context the human doesn't. And the human may have new concerns based on the report findings. This phase closes the loop and grows the skill over time.

#### The Post-Flight Conversation

The tester shares a brief summary of what was found, then asks:

> "Here's what I found this flight: [summary — key outcomes, failures investigated, bugs, notes].
>
> Based on what I saw, I think we should also test [tester's suggestion based on patterns observed].
>
> Is there anything else you'd like me to focus on next time?"

#### What the tester suggests (based on what they observed):
The tester doesn't just report — they **think ahead**. Examples:

| Tester observed... | Tester suggests... |
|--------------------|-------------------|
| 3 name cards with Format B went to review | "I should test more Format B cards — may be a pattern" |
| All CJK patients sorted correctly | "CJK is solid. I'd shift focus to multi-word Western names next" |
| Duplicate detection skipped 361 photos | "Should I re-sort with clean output to test first-run accuracy?" |
| Settings changes didn't affect filenames | "I should verify every setting against actual output files next flight" |
| Review folder had 40 photos but only 2 were real failures | "The gap threshold might be too aggressive — should I test with 2h instead of 1h?" |

#### What the human might add:
The human sees the report and thinks of new things:

- "Test with the new TEST_INPUT 2 folder I just added"
- "I changed how folder matching works — focus on existing patient folders"
- "Ignore the Settings bugs for now, I need to know if the core sort is reliable"
- "Can you check what happens when two patients have the same last name?"

#### Updating the Skill

New testing goals from the post-flight conversation get appended to `INSIDER_GOALS.md`:

```markdown
### Flight [N] Additions — [date]
Source: Post-flight Q&A

New testing goals:
- [what the human asked for]
- [what the tester suggested and human approved]

Updated priority:
- [any shift in focus for next flight]
```

**The skill grows with every flight.** After 3-4 cycles, `INSIDER_GOALS.md` becomes a rich, project-specific testing playbook that no generic checklist could replicate.

#### When to Prune

If INSIDER_GOALS.md gets long, the tester consolidates:
- Completed/resolved items move to a `### Archived` section
- Contradictory goals get resolved with the human
- Stale goals (from features that changed) get removed
- The Goal Verification Checklist gets updated to reflect current priorities

---

## Reporting Format

### OUTCOME Findings (NEW — highest priority)
Results of inspecting actual output. These are the most valuable findings.

```
OUTCOME-XXX: Title
  Lens: Goal Verification
  Dataset: which test data
  Location: where in output (Finder path, folder, file)
  Finding: what you observed
  Expected: what should have been there
  Root Cause: why this happened (if determinable)
  Severity: [Correct | Minor Inaccuracy | Wrong Result | Data Loss]
  Evidence: screenshot or file listing
```

Examples:
```
OUTCOME-001: Name card for "Tien, An Vinh" not detected
  Dataset: TEST_INPUT
  Location: _Review/2025-05-19_1430/ (photo DSC_0102.JPG)
  Finding: Name card photo with clear "Patient / Tien, An Vinh" text went to review
  Expected: Should have started a new patient series
  Root Cause: Format B detection requires "Patient" alone on line — this card has extra text
  Severity: Wrong Result
  Evidence: [screenshot of name card photo]

OUTCOME-002: 3 photos missing from output
  Dataset: TEST_INPUT 2
  Location: Not found in any patient folder or _Review
  Finding: Input had 516 photos, output has 480 sorted + 33 review = 513
  Expected: 516 = sorted + review + excluded
  Root Cause: Unknown — potential data loss
  Severity: Data Loss — CRITICAL
  Evidence: [file listing comparison]
```

### Bug Classification
Things that are broken. Clear expected vs. actual.

```
BUG-XXX [Severity]: Title
  Lens: [Works / Outcome / UX / Logic / Edge]
  Screen: where it's triggered
  Steps: how to reproduce
  Expected: what should happen
  Actual: what does happen
  Impact: how this affects the user's goal
  Output verification: did this affect the actual output? (check Finder)
```

### Notes Classification
Observations that aren't bugs but inform product decisions.

```
NOTE-XXX: Title
  Lens: [UX / Logic / Edge]
  Screen: where
  Observation: what you noticed
  Reasoning: why this matters (think it through, don't just report)
  Suggestion: what to consider changing
```

---

## Risk Map — Where to Spend Testing Time

Adapted from James Bach's Heuristic Test Strategy Model (HTSM). Before testing, the Insider builds a risk map: which product elements carry the highest risk, and which quality criteria matter most for each.

**Product Elements × Quality Criteria = Test Priority**

The Insider fills this out during Phase 0 setup (after reading the spec, before testing):

```
Example for PhotoSortVision:

| Product Element        | Accuracy | Data Safety | Performance | Usability | Risk |
|------------------------|----------|-------------|-------------|-----------|------|
| Name card detection    | CRITICAL | -           | Medium      | -         | HIGH |
| Photo grouping (gap)   | HIGH     | -           | Low         | Medium    | HIGH |
| Folder matching (fuzzy)| HIGH     | Medium      | Low         | -         | HIGH |
| File copy + duplicates | Medium   | CRITICAL    | Medium      | -         | HIGH |
| Review assignment UI   | -        | Low         | -           | HIGH      | MED  |
| Settings persistence   | Low      | -           | -           | Medium    | LOW  |
| Live Preview           | -        | -           | -           | Medium    | LOW  |
```

**How to use the risk map:**
- HIGH risk elements get tested first and deepest (Phases 1-2)
- CRITICAL quality criteria get verified against actual output, not just UI
- LOW risk elements get checked during the element audit (Phase 5), not before
- The risk map is updated after each flight based on findings

**Why AI testers need this explicitly:** Without a risk map, AI agents test breadth-first — giving equal time to every feature. Human experts test depth-first on high-risk areas. The risk map forces the AI to prioritize like an expert.

---

## Session Structure — Time-Boxing

Adapted from Session-Based Test Management (SBTM). Each testing phase should be a focused session with a charter, a time box, and a debrief.

**Why time-boxing matters for AI testers:** Without limits, AI agents can spend 2 hours clicking through Settings tabs (as happened in our PhotoSortVision audit) while spending 0 minutes checking the output folder. Time-boxing prevents this.

Each session has:
- **Charter** — one sentence describing what this session will verify
- **Time box** — maximum duration (guided by the phase timing table)
- **Debrief** — what was tested, what was found, what remains untested

```
Example session:
Charter: "Verify sort output correctness — open destination folder in Finder,
         check 5 patient folders for correct photo assignment"
Time box: 30 minutes
Debrief: Checked 5/37 folders. 4 correct. 1 had a photo from a different
         patient (DSC_0102 — name card Format B not detected). Pivoted to
         test 3 more Format B cards — 2 of 3 also failed. Pattern confirmed.
         Filed OUTCOME-001 and BUG-015.
```

The debrief feeds into the Phase 7 report and Phase 8 Q&A with the human.

---

## Anti-Patterns — What Insider Testing Is NOT

1. **Not a screenshot tour.** Screenshots of every screen saying "looks good" is not testing. You must verify outcomes.

2. **Not element bingo.** Checking "button exists ✓" without evaluating what the button actually does to the output is Cold-tier work wearing an Insider hat.

3. **Not spec compliance only.** The spec can be wrong. The spec can have redundant features. The Insider tests the *product*, not just the spec.

4. **Not UI-only.** If the app produces files — **open the files.** The UI is the means, not the end. The output folder in Finder is the ground truth.

5. **Not single-dataset.** One test run with one dataset is a demo, not a test. Use every available dataset.

6. **Not "it didn't crash."** Completing without errors is the bare minimum. The question is: did it complete *correctly*?

7. **Not "count matches."** The app says "37 patients found" and the output has 37 folders. Great. But are the RIGHT photos in the RIGHT folders? Open them and check.

8. **Not reporting without reasoning.** "Preview doesn't update" is a shallow finding. "Preview doesn't update for separator changes — but Name Order radio is redundant with token chip ordering, so that's a design issue not a bug" is an Insider finding.

9. **Not log-and-move-on.** (AI-specific trap) AI agents find an issue, log it, and proceed to the next item. Human experts find an issue and *pivot* — testing 5 related cases to understand the scope before moving on. One investigated bug that reveals a pattern is worth more than 10 surface-level observations.

---

## Goal Verification Checklist Template

This template is generated during Phase 0 (Q&A) and customized per project. The Insider runs this after every test flight.

```markdown
# Goal Verification Checklist — [Project Name]

## Primary Goal: [from Q1]
□ [Specific verification step]
□ [Specific verification step]
□ [Specific verification step]

## Output Inspection: [from Q2]
□ Open [output location] in [Finder / browser / tool]
□ Spot-check [N] items for correctness
□ Verify [specific detail] matches [setting]
□ Compare [output count] against [input count]

## Failure Investigation: [from Q3]
□ Open [failure location]
□ Count failures: ___
□ Classify each failure (correct / engine issue / bug)
□ For wrong-result failures: screenshot + root cause analysis

## Cross-Dataset Comparison: [from Q4]
□ Run with Dataset 1: results ___
□ Run with Dataset 2: results ___
□ Compare: any differences in accuracy, failures, performance?

## Known Hard Cases: [from Q5]
□ [Specific hard case 1]: result ___
□ [Specific hard case 2]: result ___
□ [Specific hard case 3]: result ___

## Settings Impact: [from Q6]
□ Change [setting] → verify output reflects change
□ Change [setting] → verify output reflects change
```

---

## Integration Notes

### Files this skill creates:
- `FLIGHT_DECK/INSIDER_GOALS.md` — generated during Phase 0 Q&A, project-specific

### Files this skill references:
- `CLAUDE.md` / `PRD.md` — full project spec
- `FLIGHT_DECK/FLIGHT_PLAN.md` — current status, prior findings
- `FLIGHT_DECK/PREFLIGHT_CHECK.md` — testing depth config

### Files that should reference this skill:
- `TEST_FLIGHT_PROTOCOL.md` — as the method for Insider tier
- `PREFLIGHT_CHECK.md` — as a testing depth option
- `PILOT_HANDBOOK.md` — training material for Test Pilot
- `AUTOPILOT.md` — so the builder knows what the Insider evaluates

### Phase timing (approximate):
| Phase | % of Total Time | What |
|-------|-----------------|------|
| Phase | % of Total Time | What |
|-------|-----------------|------|
| Phase 0 | Setup (once) | Goal Discovery Q&A with human |
| Phase 1 | 25% | Goal Verification — Dataset 1 (output folder in Finder) |
| Phase 2 | 15% | Failure Deep-Dive (review folder investigation) |
| Phase 3 | 20% | Goal Verification — Dataset 2 |
| Phase 4 | 10% | Settings Impact (change setting → verify output) |
| Phase 5 | 15% | Element Audit (5-lens) |
| Phase 6 | 5% | Edge Cases |
| Phase 7 | 5% | Report |
| Phase 8 | 5% | Evolve — Post-flight Q&A with human |

**Note: 60% of time is on outcome verification (Phases 1-3). Only 15% is on element clicking (Phase 5). This is the inverse of how most testers allocate time — and that inversion is the whole point of Insider testing.**

### The Lifecycle

```
Phase 0 (once)          Phase 1-7 (each flight)         Phase 8 (each flight)
┌─────────────┐         ┌─────────────────────┐         ┌──────────────────┐
│ Q&A with    │         │ Test → Verify →     │         │ Report findings  │
│ human       │────────▶│ Investigate →        │────────▶│ Suggest next     │
│ (6 questions)│        │ Report               │         │ Ask human        │
└─────────────┘         └─────────────────────┘         └────────┬─────────┘
      │                          ▲                               │
      │                          │                               │
      ▼                          │                               ▼
┌─────────────┐                  │                      ┌──────────────────┐
│ INSIDER_    │                  │                      │ Update           │
│ GOALS.md    │──────────────────┘                      │ INSIDER_GOALS.md │
│ (created)   │                                         │ (evolved)        │
└─────────────┘                                         └──────────────────┘

Flight 1: Goals from Q&A → Test → Report → Human adds "test folder matching"
Flight 2: Goals + folder matching → Test → Report → Human adds "test with 2h gap"
Flight 3: Goals + folder matching + 2h gap → Test → Report → Tester suggests "CJK is solid, focus on Format B"
...
INSIDER_GOALS.md becomes a project-specific testing playbook.
```

---

*Draft v0.4 — Test Pilot Loop — Insider Testing Skill*
