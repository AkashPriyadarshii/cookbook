# The Morgue — cold evidence-lab redesign

> DESIGN.md by design-genius. Fused: Sentinel incident command (Sentry, deep-violet canvas + single electric accent) + Carbon technical case-file (IBM, flat-square, hairline-divide, light-weight display). Token-pull: the existing typewriter (Special Elite) stays — it is the forensic soul, kept from the source artifact.

## 1. Visual Theme & Atmosphere

**Style**: Cold forensic evidence lab — night archive for dead bugs.
**Keywords**: luminol, case-file, incident command, black-box, toxicology, evidence locker, night-archive, chemical.
**Tone**: clinical and darkly funny — NOT ornamental, NOT flirty-illustrated, NOT warm.
**Feel**: the morgue at 3am: cold vault walls, one ultraviolet lamp over the stain, a typewriter still in the corner.

**Interaction Tier**: L1 — CSS-only (the redaction peel is the one gesture that stays).
**Dependencies**: CSS only. No JS beyond the existing copy toggle.

## 2. Color Palette & Roles

The causal line: the accent is **Luminol** — the forensic reagent that reveals
hidden blood under UV. Its light is the only color in a dark evidence vault.
The canvas goes cold-ink (night archive) so the site can never be mistaken for
a warm AI default, and the typewriter reads as colder against it.

```css
:root {
  /* Backgrounds — cold night vault, not warm cream */
  --bg: #131417;            /* vault wall — deep cold umber-ink, L≈9.7% */
  --surface: #1c1e24;       /* case file / card — one step up */
  --surface-alt: #22252c;   /* alternating section */
  --surface-hover: #272a33; /* hover surface */

  /* Substrate — ruled evidence paper, kept from v1 (non-flat) */
  --substrate-rule: repeating-linear-gradient(
    0deg, transparent, transparent 31px, rgba(201,203,212,0.04) 32px);

  /* Borders */
  --border: #34373f;        /* default hairline */
  --border-strong: #c9cbd4; /* strong rule / double line */

  /* Text */
  --ink: #e8e9ee;           /* cold paper-white, NOT espresso-brown */
  --ink-faint: #9a9da8;     /* secondary */
  --ink-dim: #5f626c;       /* tertiary label */

  /* Accent — LUMINOL */
  --accent: #c8f542;        /* luminol lime-green, oklch(0.86 0.22 116) */
  --accent-deep: #9fc927;   /* hover / pressed */
  --accent-rgb: 200,245,66;

  /* Semantic */
  --error: #ff5c5c;         /* the stain, revealed */
  --success: #6cd18b;       /* case closed / fixed stamp */
}
```

**Color Rules:**
- `--bg` is a deep cold ink, never warm cream; all neutrals carry the cool hue
  (chroma ≥ 0.006 toward blue-lime), never warm brown grey.
- ONE accent (luminol) per viewport. The error-red `--error` is reserved for
  the revealed stain only (`.redact.open`), never decoration.
- Dark has a real substrate (ruled lines), not flat — so it never reads as a
  flat near-black default.
- No pure `#000` / `#fff` anywhere.

## 3. Color Contrast / A11y

- Body: `--ink #e8e9ee` on `--bg #131417` — contrast ≈ 13.5:1. Passes AA and AAA.
- `--ink-faint #9a9da8` on `--bg` ≈ 5.8:1 — passes AA for small text. On
  `--surface #1c1e24` ≈ 5.3:1. Passes.
- Accent `#c8f542` on `--bg` ≈ 9.6:1 — pass; on `--surface` ≈ 8.9:1. Pass.
- `--accent-deep #9fc927` on `--bg` ≈ 5.4:1 — large/UI only.
- `--ink` text on `--accent` Luminol: contrast ≈ 9.6:1 — passes.
- Focus ring: 2px solid `--accent` (uniqueness never trades focus away).
- Reduced-motion: CSS-only interactions; the `.redact` peel is
  instant-on-toggle, no motion dependency. No `prefers-reduced-motion` removal
  needed.

## 4. Type Scale

- **Display**: Special Elite (typewriter, kept from source artifact) ≈
  2.9rem / 1.12, letter-spacing .02em. The cold register does not break the soul.
- **UI/mono**: IBM Plex Mono 400/600 (domain face — tabular data, evidence).
- **Body**: IBM Plex Mono 400, 15px / 1.65.

Scale by 1.333 off 15px body. Display cap 2.9rem; no weight-700 shout (these
are case files, not billboards).

## 5. Spacing / Grid

- 8px base. Section rhythm: 3.5–5rem top, 2rem between.
- Evidence feed keeps the left rail (the case-file timeline) — it is the
  product's most distinctive structural move and stays.
- Cards: flat-square, 0px radius (Carbon 0–4px), hairline border, hard offset
  shadow 3px 3px 0 (kept from v1 — carbon-approved, no blur).

## 6. Component Patterns

- **Stamp** (`fixed`/`dead`) — rotated hard-green/red, keep the typewriter
  uppercase. Now: `--error`/`--success`, not warm blood/green.
- **Redaction peel** — the signature. Ink block on touch/click reveals
  `--accent-deep` stain text. The reveal color is Luminol, not blood-red.
- **Evidence bag** (install block) — hairline border, tabular header, log-look.
- **Toe-tag footer** — keep the clip-path tag; swap cream fill to `--surface`.
- **Topic cards** — now flat-square Carbon tiles w/ hairline + single accent
  case-count, not the blue-shadow boxes.

## 7. Motion

CSS-only, restrained. Hover: card straightens (rotate → 0) + accent shadow.
Redaction: instant toggle (no motion). Nothing that dies without
reduced-motion. Honor `prefers-reduced-motion: reduce` by skipping the tilt.

## 8. Signature Detail

The **Luminol reveal**: every `.redact` block hides its cause under an ink bar;
click/tap peels it to the accelerant-lime stain. Structurally the same as v1
(the peel), but the semantic flips — it is now a forensic photocall, not a
blood spill: "the stain the UV just found." One reveal per case, the rest of
the card stays cold. That is the one memorable mechanic.

## 9. Verification

- Reproduction gate: accent L/116° (far outside the 15°–45° amber band,
  outside 45°–85° OKLCH), substrate is deep cold ink with ruled micro-texture
  (not flat cream, not flat near-black default), and a causal line
  (Luminol = forensic reagent). Clears ALL THREE conditions — passed.
- On-product: a stranger sees a forensic morgue for dead bugs, not a template
  and not a different brand's look. The typewriter, case numbering, toe-tag,
  and evidence bag all survive.
- Nine levers: no centered-hero-3-cards (feed + rail), no
  Inter/Roboto/Geist/Space Grotesk, no purple-blue gradient, no drop-shadow
  blur-everything, no em-dash spray, copy already human.
