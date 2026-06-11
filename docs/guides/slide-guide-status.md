# Slide Guide — Status / Accountability

Mode: C — Status / Accountability

Use this as the audit checklist. Use `docs/guides/rewrite-status.md` as the rewrite prompt.

A slide has three roles, each carrying its own load:

- **Title** — the RAG signal and metric verdict in one line.
- **Body** — the concrete metric, trend, or owner/date table that carries the evidence.
- **Speaker note** — delivery cue and design rationale.

## Audit rules

Each rule = **one imperative, one rationale, one example when needed**. If a rule needs more, it does not belong here.

The deck's `Audience:` header names the room; rules 5 and 12 judge against it. With no header, assume engineering managers and cross-functional stakeholders.

1. **Title must lead with a metric or RAG signal.**
   The audience reads the verdict from the title alone before opening the slide. Cap it at about 12 words and at most two lines.
   `"🟡 Inference latency −40 %; on track for Q3"` ✓  `"Model performance update"` ✗

2. **Titles alone tell the story.**
   Skim only the titles: the overall health and top drivers are readable without opening any slide. Bad news must appear in the title, not buried in the body.

3. **One idea per content slide.**
   If it can be split, split it. Save previews and transitions for agenda or bridge slides.

4. **Body favors concrete artifacts over abstract bullets.**
   Show: metric trend chart, RAG summary table, owner/due-date table, timeline. Keep prose only when it is the sharpest evidence. Point at each body element and ask "so what?" — cut anything that does not connect to the title.

5. **Anchor claims in evidence the room trusts.**
   For a management audience: RAG status, metric trend with baseline comparison, owner name, and due date. Every action item must have an owner and a due date — flag any that lack either. Bad news must be surfaced, not buried.

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
    Status reports to management use a formal, neutral register. Avoid jargon the audience cannot act on; translate technical signals into business impact.
    `"CI flake rate 18%"` (too technical) → `"CI failures blocking 2 feature branches; estimated 3-day slip if unresolved"` (actionable)

## Workflow

Draft → fact-check → audit against the rules above → ship.

## Sources

Knaflic, *Storytelling with Data* (action title, horizontal logic); Alley, *The Craft of Scientific Presentations* (assertion-evidence: one-to-two-line sentence headline); Reynolds, *Presentation Zen* (one idea, show-don't-tell, signal-to-noise); Duarte, *slide:ology* (glance test, readability); Zelazny, *Say It With Charts* (message-driven chart choice); Mayer, multimedia-learning principles (spatial contiguity, redundancy and its boundary conditions); Tufte, *The Visual Display of Quantitative Information* (data-ink) and *Envisioning Information* (visual consistency); WCAG 2.1 (SC 1.4.1 use of colour; SC 1.4.3 contrast); WebAIM contrast guidance.
