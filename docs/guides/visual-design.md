# Slide Visual Design Guide

The visual layer only — layout, color, type, decoration, background. Storyline,
wording, sourcing, and chart *selection* live in the content guide; this governs how
a finished slide looks. Apply on any deck and any renderer. Self-contained: paste as-is.

## Principle

**Design is subtraction.** Every element must encode meaning — color = category,
arrow = flow, bold = the one word that matters — or be cut. The message comes first;
styling serves it and never distorts it — a chart must not exaggerate what the data
says. Consistency beats a beautiful one-off slide. Design for the worst channel:
projected in a bright room.

## Rules

1. **One visual focal point per slide.** Mark the key takeaway as the single
   focal point — a highlight block, a highlighted bar, or one large number,
   whichever the slide already carries; don't add a box when an existing element
   already anchors the eye. Accent is one emphasis moment per slide: use exactly
   one of accent-bold, a highlight block, or an accent arrow/callout — never two.
   (Charts and tables spend accent under their own rules; a highlighted bar or
   key cell is not counted here.) Let everything else recede.
2. **Structure palette = 1 primary + 1 accent + grayscale** (≤3 hues). A categorical
   scale — diagram or chart categories — is counted separately and capped at 4. Every
   color holds one fixed meaning; reserve red for risk/negative.
3. **Color is never the only signal.** Pair it with a label, shape, or position so it
   survives colorblind viewers and a washed-out projector. Body contrast ≥ 4.5:1.
4. **One sans-serif family, ≤3 type levels** (title ~30 / body ~22 / footnote ~11).
   Tabular figures for any column of numbers. Minimum legible body ~18pt.
5. **Light background, dark text.** Dark only for title and divider slides. No
   textured or photographic backgrounds behind content.
6. **Tables stripped to data-ink.** Kill gridlines and zebra striping; one rule under
   the header. Numbers right-aligned, units in the header, the key cell highlighted.
7. **Chart hygiene.** No 3-D, no truncated baselines; label series directly instead
   of forcing a legend lookup.
8. **Diagrams carry a legend and reuse the deck's color tokens.** Keep one flow
   direction; don't invent a per-diagram palette. Give every category node a text
   label so the diagram reads without relying on fill color alone (colorblind
   viewers, a washed-out projector).
9. **Align to a grid, keep generous whitespace.** Fixed layouts, consistent margins,
   one alignment grid — a crowded slide is a failed slide. Uniform number/date/unit
   formatting. Footer = deck · date · page (+ classification if sensitive).
10. **Cut decoration.** No shadows, gradients, 3-D, clip-art, emoji bullets, or
    decorative borders. Allowed only because they carry information: the title rule,
    the highlight box, arrows, and callouts.
11. **Builds off by default** (presenter-led decks only; meaningless on a static or
    self-read deck). Use progressive reveal only for a genuinely dense diagram.

When a slide won't breathe with generous whitespace, split it across slides —
never densify to fit. Same standard for every audience.

## Appendix — tokens

Marp: set once in a `<style>` block and inherit everywhere.

```css
:root{
  --primary:#051c2c;  /* titles, rules */   --accent:#2251ff;  /* emphasis — ~5.7:1 as text */
  --ink:#1a1a1a;      --muted:#5f6671;       --risk:#d23b3b;    --bg:#fff;
}
section{font-family:Inter,"Source Sans 3",Calibri,Arial,sans-serif;font-size:22px}
h1{font-size:30px;color:var(--primary);border-bottom:2px solid var(--primary)}
strong{color:var(--accent)}                /* emphasis = weight + accent */
small{font-size:14px;color:var(--muted)}   /* footnotes / sources */
section::after{color:var(--muted)}         /* footer text via Marp `footer:` directive */
td.num{text-align:right;font-variant-numeric:tabular-nums}
.highlight{background:#e8eeff;border-left:4px solid var(--accent);padding:8px 12px}
```

Mermaid — one semantic palette, ≤4 categories, always with a legend. Pale fills
wash out under projection and read alike to colorblind viewers, so the text label
on every node is the primary cue and color is only redundant:

```
classDef mgd  fill:#fde2c4,stroke:#e8851a,color:#5a3000;  /* managed / external */
classDef core fill:#d9f2d0,stroke:#3f9e2a,color:#173d0c;  /* our compute */
classDef coord fill:#e6e0ff,stroke:#6a4cff,color:#241a66; /* coordination */
classDef sec  fill:#fde0e0,stroke:#d23b3b,color:#5a1010;  /* security / risk */
```
