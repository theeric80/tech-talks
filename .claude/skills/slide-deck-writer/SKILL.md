---
name: slide-deck-writer
description: 'Drafts, rewrites, and audits a presentation deck through a three-step pipeline (ghost deck → slide rewrite → audit). Routes first by delivery format (presenter-led vs standalone reading deck), then by content. Presenter-led covers three modes: A-Decision (decisions, proposals, alignment, incident reviews), B-Transfer (knowledge-transfer talks, tech intros, tool walkthroughs, spike reports), C-Status (recurring status reports, sprint reviews, retrospectives). Standalone covers the Reading-deck track (read-ahead docs, leave-behinds, board appendices) in decision / status / mixed flavors. Not for single-slide edits, formatting-only changes, or finished decks that need only minor copyediting.'
when_to_use: '"I need a deck on X", "help me write a presentation about Y", "draft slides for Z", "write me a talk on X", "I need slides for my presentation", "help me prepare a tech talk", "I need a status report deck", "help me write a sprint review", "I need a read-ahead deck", "a leave-behind the committee reads on their own".'
---

# Slide Deck Writer

Three-step pipeline: ghost deck → slide rewrite → audit checklist. Routes to the correct guide set first by delivery format (presenter-led vs standalone reading deck), then by content mode (Decision / Transfer / Status) — or, for a reading deck, by content flavor (decision / status / mixed).

Primary pattern: Sequential workflow orchestration with context-aware routing at Step 1.

## Out of scope

- The user wants to edit only one or two specific slides in an existing deck — handle inline.
- The user wants formatting or design changes only (colour, font, layout) — not a writing pipeline.
- The deck is already finished and needs only minor copyediting — do that directly.
- The request is for a prose document (report, spec, RFC) with no slide output. (This differs from a *standalone reading deck*, which is still slides and is in scope — see Step 1.)

## Prerequisites

- The guide files live at `docs/guides/` relative to the repo root; read them with the Read tool.
- No MCP servers required. All steps use Read + the pipeline guides.

## Step 1: Identify delivery format, then deck mode

First settle **delivery format** — it gates the rest of the pipeline:

> Will this deck be **presented live** (you narrate it) or read **standalone** (read-ahead / leave-behind, no presenter in the room)?

- **standalone** → Reading-deck track. Skip the mode question; instead pick a **content flavor** (decision / status / mixed) and proceed to Step 2 with the reading-deck guides. B-Transfer is not offered standalone.
- **presenter-led** → continue to the mode question below.

If the user's request clearly names or implies the mode, proceed to Step 2. If ambiguous, present the three options and ask the user to choose one:

> Which mode best describes your deck?
>
> - **A-Decision** — you need the audience to make a decision, approve a proposal, align on a recommendation, or review an incident. Lead with the conclusion.
> - **B-Transfer** — you are teaching the audience something new: a technology, a tool, a spike result, or an architectural concept. Lead with the Before state.
> - **C-Status** — you are reporting health: a recurring status update, sprint review, or retrospective. Lead with the RAG signal.

Route based on the user's answer:
- Decision / proposal / alignment / incident review → Mode A
- Knowledge transfer / tech intro / tool walkthrough / spike report → Mode B
- Status report / sprint review / retrospective → Mode C

## Step 2: Run the draft guide

Read and follow the guide for the chosen mode. The guide will ask for missing inputs before producing a ghost deck.

- **A-Decision** → read and follow `docs/guides/draft-decision.md`
- **B-Transfer** → read and follow `docs/guides/draft-transfer.md`
- **C-Status** → read and follow `docs/guides/draft-status.md`
- **Reading-deck** → read and follow `docs/guides/draft-reading-deck.md`

Validation gate before proceeding to Step 3:
- A-Decision: title-only readthrough must tell the full story top-down (Pyramid Principle).
- B-Transfer: title sequence must follow Before/After → Concept → Demo → Adopt order; Slide 1's speaker note must carry the empowerment promise ("After this talk, you'll be able to [X]"); the final slide must restate that capability as the take-home.
- C-Status: title-only readthrough must reveal health signal and top drivers without opening any slide.
- Reading-deck: title-only readthrough must tell the story top-down AND Slide 1 must read as a self-contained executive summary (shaped to its content flavor).

If the gate fails, iterate on the ghost deck until it passes. State the failing test explicitly and show the fix.

## Step 3: Run the rewrite guide

Read and follow the slide-level rewrite guide. It polishes each slide stub from Step 2 into a finished slide.

- **A-Decision** → read and follow `docs/guides/rewrite-decision.md`
- **B-Transfer** → read and follow `docs/guides/rewrite-transfer.md`
- **C-Status** → read and follow `docs/guides/rewrite-status.md`
- **Reading-deck** → read and follow `docs/guides/rewrite-reading-deck.md`

Rewrite every slide in the ghost deck in order. Do not skip slides. After rewriting, confirm that title-level logic still holds (same test as the Step 2 gate).

## Step 4: Run the audit guide

Read and follow the audit checklist for the chosen mode. This is the final quality gate.

- **A-Decision** → read and follow `docs/guides/slide-guide-decision.md`
- **B-Transfer** → read and follow `docs/guides/slide-guide-transfer.md`
- **C-Status** → read and follow `docs/guides/slide-guide-status.md`
- **Reading-deck** → read and follow `docs/guides/slide-guide-reading-deck.md`

Work through the checklist slide by slide. Report each finding with the slide number and the rule or check violated. Apply fixes immediately. When all checklist items pass, the deck is done.

## Stop criteria

The pipeline is complete when:
1. The ghost deck passed its mode-specific title readthrough test (Step 2).
2. Every slide has been rewritten per the rewrite guide (Step 3).
3. Every checklist item in the audit guide passes with no open findings (Step 4).

## Examples

### Example 1: Decision deck

User says: "I need a 15-min deck recommending we migrate from Postgres to a time-series DB."

Actions:
1. Identify mode: "migrate … recommending" → A-Decision.
2. Read `docs/guides/draft-decision.md`. Gather Purpose / Audience / Context / Success criteria; produce ghost deck.
3. Run title-only readthrough — confirm story holds top-down.
4. Read `docs/guides/rewrite-decision.md`. Rewrite each slide.
5. Read `docs/guides/slide-guide-decision.md`. Run audit; fix any findings.

Result: A finished decision deck, audit-clean.

### Example 2: Knowledge-transfer talk

User says: "Help me write a 20-min internal talk introducing Go structured concurrency."

Actions:
1. Identify mode: "introducing … talk" → B-Transfer.
2. Read `docs/guides/draft-transfer.md`. Gather framing inputs; produce ghost deck with Before/After → Concept → Demo → Adopt structure.
3. Validate sequence order in titles, the Slide-1 empowerment promise, and the closing take-home slide.
4. Read `docs/guides/rewrite-transfer.md`. Rewrite each slide.
5. Read `docs/guides/slide-guide-transfer.md`. Run audit; fix any findings.

Result: A finished knowledge-transfer deck, audit-clean.

### Example 3: Status report

User says: "I need a sprint review deck for Sprint 42. Velocity is on track but CI is broken."

Actions:
1. Identify mode: "sprint review" → C-Status.
2. Read `docs/guides/draft-status.md`. Gather framing inputs; produce ghost deck with Health → Drivers → Actions structure.
3. Validate titles surface health signal and drivers without opening slides.
4. Read `docs/guides/rewrite-status.md`. Rewrite each slide.
5. Read `docs/guides/slide-guide-status.md`. Run audit; fix any findings.

Result: A finished status deck, audit-clean.

### Example 4: Standalone reading deck

User says: "I need a read-ahead deck for the steering committee — our H2 cost-optimization plan plus current program status. They read it before the meeting; I won't present it."

Actions:
1. Identify delivery format: "read it before the meeting … won't present" → standalone → Reading-deck track. Content flavor: recommends *and* reports → mixed.
2. Read `docs/guides/draft-reading-deck.md`. Gather framing inputs; produce ghost deck with Slide 1 as a combined SCR-led executive summary, TOC + section dividers, backup detail in an appendix.
3. Validate title-only readthrough holds and Slide 1 reads self-contained.
4. Read `docs/guides/rewrite-reading-deck.md`. Rewrite each slide self-sufficient, no speaker note relied upon.
5. Read `docs/guides/slide-guide-reading-deck.md`. Run audit incl. the deck-level checks; fix any findings.

Result: A finished standalone reading deck, audit-clean.

## Troubleshooting

### Title-only readthrough fails at Step 2

**Cause:** Storyline sentences were not truly sequential — a later slide does not follow causally from the previous one, or the opening slide does not carry the BLUF (A-Decision) / Before state (B-Transfer) / health signal (C-Status) / executive summary (Reading-deck).

**Solution:** Return to the Storyline section of the draft guide. Rewrite the offending sentences as full causal claims, then regenerate the slide stubs. Rerun the readthrough before proceeding.

### User's request matches two modes

**Cause:** For example, a post-incident review that also teaches the team a new runbook pattern.

**Solution:** Ask the user: "Is the primary goal to get a decision / approval, or to teach the team something?" Route to the mode that matches the primary goal. The secondary concern becomes a supporting slide, not a mode switch.

### A guide file is not found

**Cause:** The repo root or guide directory path has changed.

**Solution:** Run `find . -name "draft-decision.md"` from the repo root to locate the guides. Update the path and proceed.
