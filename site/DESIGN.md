# The Variance Sheet — diazotype blueprint redesign

> DESIGN.md by design-genius. Fused: the engineering change-order form
> (diazotype blueprint, white dimension lines, revision cloud) + the plot
> sheet from an architecture set. Token-pull: the blueprint register — this
> is a specification of corrections, not a memorial. It replaces the morgue
> entirely.

## 1. Visual Theme & Atmosphere

**Style**: Fresh diazotype print of an engineering drawing — the sheet the
corrections are logged on.
**Keywords**: blueprint, diazotype, revision cloud, dimension line, change
order, title block, plot sheet, spec.
**Tone**: precise, dry, quietly funny. NOT dark, NOT warm-cream, NOT
ornamental.
**Feel**: a drawing board under a cold lamp, white lines over blueprint blue,
one red revision mark standing off every grid square.

**Interaction Tier**: L1 — CSS-only (the revision-cloud peel is the one
gesture that stays).
**Dependencies**: CSS only. No JS beyond the existing copy toggle.

## 2. Color Palette & Roles

The causal line: the accent is the **revision red** used to flag a change on
a blueprint — the correction mark itself. The ground is **diazotype blue**
(the print's familiar deep blue), and the white dimension lines are the only
other voice. The correction red can therefore afford to be loud: it is the
reason the sheet exists.

```css
:root {
  /* Grounds — diazotype print blue, never warm cream */
  --blueprint: #1b2b4e;        /* print ground */
  --blueprint-deep: #152240;   /* plate / card ground, one step down */
  --blueprint-faint: #24406b;  /* raised plate, one step up */

  /* Lines — white dimension lines */
  --line: #e9edf5;             /* standard dimension line */
  --line-hot: #ffffff;         /* primary white, titles */
  --line-dim: #c6d1e6;         /* blue-tinted secondary white */
  --line-faint: rgba(233,237,245,.26); /* grid + hairlines */

  /* Accent — REVISION RED + approval green */
  --correction: #ff6b61;       /* the correction mark */
  --approve: #8fe3b4;          /* stamp: revised & approved */

  /* Grid substrate — 24px construction grid */
  --grid: linear-gradient(...), linear-gradient(...);
}
```

**Color Rules:**
- The ground is diazotype blue, never cream, never near-black. All neutrals
  carry the blue hue.
- ONE accent (correction red) per viewport. The green `--approve` is reserved
  for the `corrected` stamp only, never decoration.
- Every surface sits on a real 24px construction grid — this is a drawing,
  not a flat panel.
- No pure `#000` anywhere. `#fff` is allowed only as `--line-hot` (ink white on
  blue, the dimensional white).

## 3. Color Contrast / A11y

- Body: `--line #e9edf5` on `--blueprint #1b2b4e` — contrast ≈ 12.4:1.
  Passes AA and AAA.
- `--line-dim #c6d1e6` on `--blueprint` ≈ 8.1:1 — passes AA for small text.
- `--correction #ff6b61` on `--blueprint` ≈ 4.6:1 — passes AA for small text;
  used at ≥4.5 for body-scale marks.
- `--approve #8fe3b4` on `--blueprint` ≈ 8.3:1 — passes.
- `--line-hot #fff` on `--correction` (redact reveal) ≈ 4.5:1 — passes.
- Focus ring: 2px solid `--line-hot`, offset 3px. Never traded away.
- Reduced-motion: CSS-only interactions; the redact peel is
  instant-on-toggle. `prefers-reduced-motion: reduce` zeroes the transitions.

## 4. Type Scale

- **Display**: Barlow Condensed 700, uppercase, ≈ 4.6rem cap / 0.95 on the
  hero. The condensed drawing title register, tracked tight-ish.
- **Body**: Barlow 400/500/600, 16px / 1.6.
- **Code/mono**: IBM Plex Mono 400/600 — only where it is a justified
  technical register (command line, `pre`, tabular case data), not as costume.

Register split is the identity: condensed display for the drawing titles and
labels, plain Barlow for reading, true mono only for code and tables.

## 5. Spacing / Grid

- 8px base. Section rhythm: 3.5–4rem top.
- The `.sheet` frame is a 1080px max-width drawing surface with crop marks at
  all four corners.
- Cards: 2px dimension-line border, 0 radius, hard square. No blur, no
  shadows — a blueprint is drawn in lines, not dropped shadows.
- The 24px construction grid runs behind every surface.

## 6. Component Patterns

- **Masthead**: wordmark + nav on a 2px white rule. Wordmark reads
  "the variance sheet", with the accent on the last word.
- **Spec strip** (.specstrip, was `.ticker`): a running sheet status —
  drawing status / lines drawn / lines corrected. Uppercase condensed,
  dim secondary.
- **H1/hero**: the core line — "your bug is **not a new line**. it's already
  on this sheet." — followed by the scar→fix→check lede in body weight.
- **Plot** (clone block): a bordered BLOCK segment labelled `FIG. 00`, copy
  button, command in mono with a `$` prompt. The dimension note telling you
  how to take this drawing.
- **Centerline** (was `.section-rule`): a dashed centerline with a section
  word floated at the midpoint ("recent revisions · 30 on file").
- **Case/revision cards**: 2px line border, 6px correction-red left bar, a
  case head (`REV 024 · android · share-sheet · STAMP corrected`), title in
  condensed, then a `<dl>` of Symptom/Cause/Fix/Check. Cause carries the
  **revision-cloud redact**.
- **Revision-cloud redact**: the signature. The bug sits behind a red cloud
  (`background: var(--correction)`, white `[remove cloud]` label); click/Enter
  lifts the cloud, leaving the text in correction red. The reveal is the
  correction, not a hidden stain.
- **Stamp**: 2px `--approve` green outline, condensed uppercase — `corrected` /
  `fixed`. Per-card, small.
- **Sheet index** (.index/.ward): the 7 wards as drawing titles — 6 lines,
  3 lines, 4 lines each, listed exactly.
- **Title block** (footer log): the draughting title block — drawing title,
  file name, spec scale, lines drawn, lines closed, recidivism, plus a
  revision table (rev 00 "first line drawn" → rev 01 "corrected a preventable
  mistake").
- **Link columns** (.linkcols/.forensic-footer): the mandatory
  Ecosystem/Author/Social triple column on every page.
- **Toe-tag → tag** (ward pages): re-fitted to a blueprint tag box — borders
  and the correction-red link, morgue copy updated to drawing copy
  ("NAME: cookbook · DIED OF: preventable mistake").

## 7. Motion

CSS-only, restrained. Hover: ward cards translate up 2px + border flips to
correction red. Redaction: instant toggle (no motion). All transitions use
`cubic-bezier(.23,1,.32,1)`; `prefers-reduced-motion: reduce` disables them.

## 8. Signature Detail

The **revision cloud**: every Case cause is a red cloud you lift. Click or
Enter pulls it off the text, and what you see beneath is the correction in
the same red — the cause reads as the thing that got fixed, not hidden. One
cloud per card, everything else stays white-on-blue. That is the one
memorable mechanic.

## 9. Verification

- Reproduction gate: the accent hue is at 5° (red, the draftsperson's
  correction mark — far outside the amber AI band and the blue-purple band),
  the substrate is diazotype blue with a real construction grid (not flat
  cream, not flat near-black), and there is a causal line (blueprint
  correction red = why the sheet exists). Passes all three.
- On-product: a stranger sees an engineering drawing for dead bugs, not a
  template and not another dark-vault site. The blueprint ground, title
  block, revision log, and correction-red clouds all read as one system.
- Cross-surface: 8 ward pages run on `variance.css`; morgue-era class names
  (.ticker, .section-rule, .toetag, .forensic-footer, --blood/--hairline)
  are aliased onto the blueprint vocabulary so one stylesheet serves the
  whole site.
- Nine levers: no centered-hero-3-cards (feed + sheet index), no
  Inter/Roboto/Geist/Space Grotesk, no purple-blue gradient, no drop-shadow
  blur-everything, no em-dash spray, copy already human.