# Slide Guide

Use this as the audit checklist. Use `technical-writing-rewrite-guide.md` as the rewrite prompt.

A slide has three roles, each carrying its own load:

- **Title** — the take-away in one line.
- **Body** — the concrete artifact or tight evidence that carries the idea.
- **Speaker note** — delivery cue and design rationale.

## Audit rules

Each rule = **one imperative, one rationale, one example when needed**. If a rule needs more, it does not belong here.

1. **Title states the take-away, not a label.**
   `Today: humans gate the routine` ✓  `Current flow` ✗

2. **One idea per content slide.**
   If it can be split, split it. Save previews and transitions for agenda or bridge slides.

3. **Body favors concrete artifacts over abstract bullets.**
   Show: image, code, diagram, number, comparison, workflow. Keep prose only when it is the sharpest evidence.

4. **Anchor technical claims in evidence.**
   Senior engineers look for proof and constraints. Use a metric, source, code path, trade-off, or explicit assumption.

5. **Code and diagrams are excerpts, not dumps.**
   Show the smallest slice that proves the point. Prefer editable text over screenshots; highlight the relevant line, boundary, or flow.

6. **Cut anything the visual already says or the speaker will say out loud.**
   Duplicated signal is noise. Rationale belongs in the speaker note, not the body.

7. **Readable from the back row.**
   Large text. WCAG AA contrast requires at least 4.5:1 for normal text and 3:1 for large text. Do not rely on colour alone — red-green colour blindness can affect up to about 8% of men, varying by population.

8. **Labels touch the object they label.**
   Distance forces correspondence work that costs comprehension. (Mayer, spatial contiguity.)

9. **Same word for the same thing. One diagram idiom throughout.**
   Drift makes the audience wonder whether you mean something different.

## Workflow

Draft → fact-check → audit against the rules above → ship.

## Sources

Knaflic, *Storytelling with Data* (action title); Reynolds, *Presentation Zen* (one idea, show-don't-tell, signal-to-noise); Duarte, *slide:ology* (glance test, readability); Mayer, multimedia-learning principles (spatial contiguity, redundancy); Tufte, *Envisioning Information* (visual consistency, data-ink); WCAG 2.1 (SC 1.4.1 use of colour; SC 1.4.3 contrast); WebAIM contrast guidance.
