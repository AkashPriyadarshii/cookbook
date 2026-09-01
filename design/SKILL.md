---
name: anti-default-gates
description: >
  Design/UX lesson: an anti-default ("don't use X color/pattern") gate must
  target the default reproduction AS A WHOLE, never a single element or hue
  family; and passing one gate must be necessary-not-sufficient so a generic
  alternative can't slip past. Use when writing anti-slop/sunset/hard-ban
  rules for a design skill or style guide.
---

# Anti-Default Gate Lessons

From a hard audit (design-genius) where 4 independent review lanes flagged a
well-intentioned orange-ban as color-biased. Each one broke something real.

## 1. Ban the reproduction, never the hue

A rule "no orange accent" bans a color family outright — a heritage gold on
warm ivory with a real derivation line (the gilt stamp on the product) is a
legitimate design, and force-coloring it off is the bias. It also can't tell
blanket harm from a justified use.

**Fix:** gate the default REPRODUCTION as a whole, all conditions AND-ed:
fails only if (a) accent in the banned band AND (b) flat default bg AND (c) no
causal derivation line. Clears if it escapes ANY one. Never ban a hue family.

**Check:** state the 3 conditions AND-ed; a correct use must clear it. If the
gate says "never warm on cream," it's a hue ban, not a reproduction gate.

## 2. One gate passing ≠ the audit passed

A color gate only clears the COLOR axis. A site that passes ("generic blue,
not in the banned band") can still fail type, layout, copy, motion — and a
checklist that says "color-clear → mark DISTINCT, stop" lets the audit
short-circuit before the other levers run. Silent false-negative.

**Fix:** make every gate's pass NECESSARY, NOT SUFFICIENT for the overall
verdict. "Cleared the color gate" means only that horizontal check passed; run
the full dimension rubric, and mark done only when every axis passes.

**Check:** does any gate's pass line end in "…and stop"? If yes, it can
short-circuit the audit. Every gate must say what it clears and what remains.

## 3. Bless-with-variety, not one exemplar

A gate that names ONE passing example ("gold on ivory is DISTINCT") quietly
blesses that example as the new safe default — the corpus re-converges on the
crowned pass case instead of the banned one.

**Fix:** vary the exemplar across archetypes/hue families in the SAME clause
(cool cobalt, green chlorophyll, warm gilt) so no single look is the blessed
default, and make the derivation the thing that clears — not the hue.

**Check:** does the gate's pass-example repeat one fixed swatch every time?
Diversify it.

## 4. Medium-aware tells

A web anti-slop checklist (centered hero, CTA line-wrap, hover, viewport) is
nonsense on a physical/print/industrial surface with no cursor or scroll.
Judging a print poster by "hover-only" is a false fail.

**Fix:** scope each tells-list to its medium. Web list → screen only; physical
medium → its own form rules (material, shape, substrate). Never apply one
surface's tells to another.

**Check:** would this tell even exist on a print poster or a device? If not,
label it web-only.
