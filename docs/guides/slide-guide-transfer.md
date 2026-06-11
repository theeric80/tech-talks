# Slide Guide — Knowledge Transfer

Mode: B — Knowledge Transfer

Use this as the audit checklist. Use `docs/guides/rewrite-transfer.md` as the rewrite prompt.

A slide has three roles, each carrying its own load:

- **Title** — topic label or take-away, depending on slide type (see Rule 1).
- **Body** — the concrete artifact or working example that carries the idea.
- **Speaker note** — delivery cue and design rationale.

## Audit rules

Each rule = **one imperative, one rationale, one example when needed**. If a rule needs more, it does not belong here.

The deck's `Audience:` header names the room; rules 5 and 12 judge against it. With no header, assume peer software engineers and system architects.

Three deck-level checks run once, before the per-slide rules:

**Opening check — Slide 1 speaker note contains an empowerment promise.**
Slide 1 must have a Speaker note with a second-person future-tense promise matching the Success criteria ("After this talk, you'll be able to [X]"). If the note is absent or states a topic rather than a capability, this check fails.
`Speaker note: After this talk, you'll replace duplicated type-specific helpers with one generic function.` ✓
`Speaker note: Introduction to Go generics.` ✗

**Adopt check — the Adopt phase names at least one specific pitfall.**
At least one slide or speaker note in the Adopt phase must name a specific pitfall or known limitation, not just the adoption path. A generic "it has some edge cases" does not pass; name the edge case.

**Closing check — the final slide restates the take-home capability.**
The last content slide answers the Slide 1 promise: it restates the Success criteria as a capability the audience now has, printed in the title or body because it stays up during Q&A. A deck ending on future work, "Questions?", or "Thank you" fails this check.
`"You can now replace WaitGroup patterns with errgroup"` ✓
`"Thank you! Questions?"` ✗

1. **Apply the two-case title rule.**
   A slide is a **claim slide** if its title asserts a measurable or falsifiable outcome — take-away title required. A slide that introduces a concept, section, or navigation step is a **concept/navigation slide** — descriptive title acceptable. Either way, cap the title at about 12 words and at most two lines.
   - Wrong: `"Batching architecture"` ✗ (vague label, no outcome)
   - Right (claim): `"Batching cuts p99 write latency by 40%"` ✓
   - Right (concept): `"How batching works"` ✓

2. **Titles alone tell the story.**
   Skim only the titles: the concept sequence is readable and each slide leads to the next. The Before state should precede the After state; Concept should precede Demo; Demo should precede Adopt.

3. **One idea per content slide.**
   If it can be split, split it. Save previews and transitions for agenda or bridge slides.

4. **Body favors code blocks and diffs over abstract bullets.**
   Show: working code, before/after diff, command output, diagram. Keep prose only when it is the sharpest evidence. Charts are secondary to executable artifacts. Point at each body element and ask "so what?" — cut anything that does not connect to the title.

5. **Anchor claims in evidence the room trusts.**
   For a peer engineering audience: working code, before/after diff, or reproduction steps. A claim without a runnable or reproducible artifact is weak.

6. **Code and diagrams are excerpts, not dumps.**
   Show the smallest slice that proves the point. Prefer editable text over screenshots; highlight the relevant line, boundary, or flow.

7. **Chart form follows the message.**
   Trend → line; ranking → bar; composition → stacked; correlation → scatter. If the slide names no chart type, skip this check — do not infer one.

8. **Cut anything the visual already says; in a live talk, also cut what the speaker will say out loud.**
   Duplicated signal is noise. Rationale belongs in the speaker note, not the body. Exception: keep unfamiliar or technical terms on the slide even when spoken — for new vocabulary and non-native-language audiences, that redundancy aids comprehension (Mayer's redundancy boundary condition).

9. **Readable from the back row.**
   Large text. WCAG AA contrast requires at least 4.5:1 for normal text and 3:1 for large text. Do not rely on colour alone — red-green colour blindness can affect up to about 8% of men, varying by population.

10. **Labels touch the object they label.**
    Distance forces correspondence work that costs comprehension. (Mayer, spatial contiguity.)

11. **Same word for the same thing. One diagram idiom throughout.**
    Drift makes the audience wonder whether you mean something different.

12. **Register matches the room.**
    Knowledge-transfer talks among peers use a lighter, more conversational register than formal management reviews. A title that works in a team tech talk may be too casual for a cross-org presentation.

## Workflow

Draft → fact-check → audit against the rules above → ship.

## Sources

Knaflic, *Storytelling with Data* (action title, horizontal logic); Alley, *The Craft of Scientific Presentations* (assertion-evidence: one-to-two-line sentence headline); Winston, *How to Speak* (MIT OCW) (empowerment promise); Ernst, *How to give a technical presentation* (conclusions-slide close, pacing); Reynolds, *Presentation Zen* (one idea, show-don't-tell, signal-to-noise); Duarte, *slide:ology* (glance test, readability); Zelazny, *Say It With Charts* (message-driven chart choice); Mayer, multimedia-learning principles (spatial contiguity, redundancy and its boundary conditions); Tufte, *The Visual Display of Quantitative Information* (data-ink) and *Envisioning Information* (visual consistency); WCAG 2.1 (SC 1.4.1 use of colour; SC 1.4.3 contrast); WebAIM contrast guidance.
