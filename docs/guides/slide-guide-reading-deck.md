# Slide Guide — Reading Deck / Leave-Behind

Mode: Reading-deck — standalone, no presenter

Use this as the audit checklist. Use `docs/guides/rewrite-reading-deck.md` as the rewrite prompt.

A reading deck has no oral layer. Each slide carries its full load on the page:

- **Title** — the take-away in one line; the titles alone carry the argument.
- **Body** — the concrete artifact or tight evidence, self-sufficient without narration.

There is no speaker-note role here — anything essential must be on the slide.

## Deck-level checks

Run these once, before the per-slide rules.

**Executive-summary check.** Slide 1 is a self-contained executive summary, readable in isolation, shaped to its flavor:
- decision → one-page SCR/SCQA (situation + complication + recommendation + rationale + key risks + decision needed);
- status → a one-line governing assessment above a health rollup (RAG + metrics + milestones + risks);
- mixed → one combined SCR-led summary with the health rollup subordinated.
A flat bullet dump that drops the situation/complication setup, a bare dashboard with no governing verdict, or a mini-table-of-contents that enumerates what each section says instead of synthesizing one governing thought, fails this check. State the recommendation's payoff as an audience-level outcome, not a restatement of its mechanism.

**Standalone-readability check.** Cover everything but each slide's title and body — can the page be fully understood with no narration? If any slide depends on a speaker to complete it, it fails.

**Navigation check.** For any deck past ~8 slides: a table of contents is present, section dividers reappear with the current section marked, and pages are numbered. The deck must self-orient.

**Source-citation check.** Every data point carries a source. No unsourced number.

**Appendix-separation check.** Backup detail — financials, methodology, scenarios — lives in an appendix, not in main-line body pages.

## Audit rules

Each rule = **one imperative, one rationale, one example when needed**. If a rule needs more, it does not belong here.

The deck's `Audience:` header names the room; rules 5 and 12 judge against it. With no header, assume experienced software engineers and system architects.

1. **Title states the take-away in one short line, not a label.**
   Cap it at about 12 words and at most two lines — past that a title stops being glanceable.
   `H2 plan closes the 13% remaining overrun` ✓  `Cost plan` ✗

2. **Titles alone tell the story.**
   Skim only the titles: the argument holds and each leads to the next. For a reading deck this is not optional — title-skimming is how the reader navigates a deck no one presents.

3. **One idea per content slide.**
   If it can be split, split it. Self-sufficiency is bought with more pages, not denser pages. Save previews and transitions for the agenda or section dividers.

4. **Body favors concrete artifacts over abstract bullets.**
   Show: table, number, comparison, diagram, workflow. Keep prose only when it is the sharpest evidence. Point at each body element and ask "so what?" — cut anything that does not connect to the title.

5. **Anchor claims in evidence the room trusts.**
   Different rooms trust different proof — engineers weigh metrics, code paths, and trade-offs; a management or executive review weighs costs, owners, risks, verification results, and their sources.

6. **Code and diagrams are excerpts, not dumps.**
   Show the smallest slice that proves the point. Prefer editable text over screenshots; highlight the relevant line, boundary, or flow. Full detail goes to the appendix.

7. **Chart form follows the message.**
   Trend → line; ranking → bar; composition → stacked; correlation → scatter. If the slide names no chart type, skip this check — do not infer one.

8. **The oral layer migrates onto the slide as bounded structure, not prose.**
   A live deck cuts what the speaker says aloud; a reading deck has no speaker, so that content must appear on the page — as a labeled takeaway box, annotation, or footnote, never as added paragraphs. Still cut anything the visual already says: duplicated *visual* signal is noise. Exception: keep unfamiliar or technical terms on the slide even when redundant — for new vocabulary and non-native-language audiences, that redundancy aids comprehension (Mayer's redundancy boundary condition).

9. **Legible on screen and in print.**
   A reading deck is read on a laptop or printed, not from the back row — but small type and thin contrast still fail. WCAG AA contrast requires at least 4.5:1 for normal text and 3:1 for large text. Do not rely on colour alone — red-green colour blindness can affect up to about 8% of men, varying by population.

10. **Labels touch the object they label.**
    Distance forces correspondence work that costs comprehension. (Mayer, spatial contiguity.)

11. **Same word for the same thing. One diagram idiom throughout.**
    Drift makes the reader wonder whether you mean something different — and there is no speaker to clarify.

12. **Register matches the room.**
    The same take-away wears different voices — punchy for an engineer audience, formal for an executive review; a title that lands in one reads flippant in the other. A reading deck for senior leadership uses a formal, neutral register.
    `We torched 22% of the budget on idle clusters` (too casual) → `Idle staging clusters drove a 22% compute overrun` (executive register)

## Workflow

Draft → fact-check → audit against the deck-level checks and the rules above → ship.

## Sources

Minto, *The Pyramid Principle* (SCQA/SCR, governing thought, MECE, top-down); Knaflic, *Storytelling with Data* (action title, horizontal logic); Alley, *The Craft of Scientific Presentations* (assertion-evidence: one-to-two-line sentence headline); Reynolds, *Presentation Zen* (one idea, show-don't-tell, signal-to-noise); Duarte, *slide:ology* (synthesis, glance test, readability); Zelazny, *Say It With Charts* (message-driven chart choice); Mayer, multimedia-learning principles (spatial contiguity, redundancy and its boundary conditions); Tufte, *The Visual Display of Quantitative Information* (data-ink) and *Envisioning Information* (visual consistency); MBB practice (executive summary built last, detail-to-appendix separation, source discipline, standalone self-sufficiency); WCAG 2.1 (SC 1.4.1 use of colour; SC 1.4.3 contrast); WebAIM contrast guidance.
