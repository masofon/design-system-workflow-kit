---
name: ds-design-journey
description: Design-side workflow for building and linting SCREENS, JOURNEYS and prototypes in Figma (as opposed to library components) — page/screen setup, driving many screens from preview components, the chrome-vs-specimen rule, and a repeatable lint pass covering text styles, sentence-case content, gap/padding token binding, and real-component usage. Use when designing or auditing a multi-screen flow, wireframe or interactive prototype, when asked to "lint" a screen/journey file, or when text is showing random fonts / spacing looks off.
---

# Design journey (Figma / screens & flows)

Companion to `ds-design-component` (which owns *library* components). This skill owns **screen-
and journey-level** files: multi-screen flows, wireframes, teardowns, prototypes.

File keys and the token model are in config § Paths → `design_foundations`. Requires `figma-use`
for all reads and writes.

## 0. Before touching anything

- **Inspect before you edit.** Read the real property values (`layoutMode`, `itemSpacing`,
  padding, `boundVariables`, `textStyleId`, resolved `fontName`). Screenshots verify; they don't
  tell you intent.
- **Find the main components first.** Screen files are usually driven by a handful of page-level
  COMPONENTs instanced across every screen. Fixing a main propagates to all instances for free;
  fixing instances one by one creates overrides and wasted work.
  `pg.findAllWithCriteria({types:['COMPONENT']})` plus a scan for repeated structure finds them.
- **Establish the reference.** If the person says "I fixed this one, match the rest", read the
  fixed node's full property set and diff it against its siblings. Derive the settings from the
  artefact, don't guess them. Report the diff before applying.
- **Exclude imported reference material.** These files often contain pasted-in screens from
  other UI kits or older libraries, kept as visual reference. They use foreign styles and are
  not ours to lint. Identify and skip them; say which you skipped.

### Odd geometry is deliberate until proven otherwise

Static prototypes fake motion and state using offsets, overflow, clipping and alignment — those
are the only tools available. So geometry that looks broken is frequently the design. **Never
bundle it into a "fixed N pre-existing bugs" list; report it and ask.**

Treat as deliberate until confirmed: absolute positioning inside auto-layout, negative
coordinates, content wider than its container, clipped edges, non-`MIN` alignment, and above all
**per-screen variation in an otherwise identical component**.

The tell is *pattern across the sequence*. If a value changes monotonically with step or screen
order, it encodes state. Real bugs are uniform and pointless (text clipped in a fixed-width label
column, a container narrower than its own text); authored states are varying and meaningful. Two
different mechanisms achieving the same visual effect across a sequence (one screen
absolute-positioned, three others `MAX`-aligned) is still intent, not inconsistency.

**Check the second-order effect of every resize.** Changing a container's width silently
recalibrates every offset inside it. Compute the travel available to any overflowing child before
and after — `child.width − parent.innerWidth` — and flag the delta. Widening a panel 600→704 once
cut a step rail's travel from 158px to 54px and made an authored `x=-118` overshoot its end by
96px, quietly destroying a progressive-reveal interaction. Surface that trade-off **before**
applying the resize, not after.

**Layout rules** for every screen and prototype: `design_foundations` § Building layouts.

## 1. The chrome-vs-specimen rule (most important concept here)

Journey files that showcase branding contain two kinds of text and visual, and they must be
treated **oppositely**:

- **Chrome** — the tool's own UI: panel headings, card titles, row labels, readout values, helper
  copy, button labels. Chrome gets design-system styles and tokens, always.
- **Specimen** — content that exists to demonstrate a *customer's* or sub-brand's look: logo
  lockups, type samples ("Aa Bb Cc 123", "The quick brown fox"), font-picker "Ag" swatches, mono
  hex/token readouts. Specimens deliberately carry raw fonts and values. Styling them destroys
  the thing they exist to show.

Identify specimens by **role and container** (`.../lockup/mark`, a type-sample row, a swatch's
hex caption), **never by font**. A brand font on a chrome label is the bug you're fixing; the same
font on a specimen is the point. When genuinely unsure, leave it and flag it — a wrongly-styled
specimen is a silent regression.

## 2. Text rules

### 2.1 Content follows config § Writing — transforms come from the style

**Never bake capitalisation into text content.** Author every label, value and heading in the
case config § Writing → `text_case` sets (sentence case by default: "Base logo", "Brand
guidelines"). Uppercase, Title Case and lowercase are presentation, applied by the text style's
`textCase`, so they can be re-themed and the content stays readable and translatable.

Auditing this — check each style's `textCase` in `design_foundations` § Text styles:

- **The style already transforms** (`textCase: UPPER`, typically an overline style). Converting
  `"TYPOGRAPHY"` → `"Typography"` on a node using it is a **zero-visual-change** fix — do these
  without asking.
- **The style doesn't transform** (`textCase: ORIGINAL`, typically labels). Converting baked caps
  there **visibly** changes the label. That's a design decision, not a lint: either (a) the label
  should read uppercase → move it to the uppercase style, or (b) it should read in sentence case
  → convert the content and keep the style. **Report the list and let the person choose**; don't
  bulk-convert.
- Legitimate exceptions to leave alone: config § Writing → `case_exceptions` (acronyms, hex
  values, single-glyph specimens by default).

### 2.2 Every text node carries a text style

If the text styles bind font properties to variables (`design_foundations` § Text styles), one
style renders different fonts in different theme contexts. Consequences:

- You never pick a font. Apply the right style and the font follows the brand and theme.
- So **raw-font text is almost always an unstyled-node bug**, except for specimens (§ 1).
- Map by **role first, size second**. Match against a nearby already-styled node playing the
  same structural role; fall back to the resolved size ramp in `design_foundations` § Text
  styles.
- **A missing monospace or code style** (check § Text styles) means deliberately-mono text (hex
  values, token names) can't be styled without losing the mono affordance — leave it raw and flag
  it.

### 2.3 Text mutation gotchas (battle-tested)

- **`setTextStyleIdAsync` changes the node's resolved font.** Any font-dependent write *after* it
  (restoring `textDecoration`, `textCase`, `characters`) throws `Cannot write to node with
  unloaded font` unless you load the NEW font. Load the style's resolved font, not just the
  node's old one.
- **Applying a style flattens mixed formatting.** A paragraph containing an inline link
  (different fill + underline) loses the underline. Capture `textDecoration`/`textCase` first,
  re-apply after, and load fonts for the post-style font. If the node is mixed *by design* (an
  inline-link demo) and you're only linting, skip it and flag it.
- **Rewriting a mixed-format paragraph (the moment to also style it).** When the copy is changing
  anyway, rebuild the ranges deliberately rather than skipping the node:
  1. Read the existing segments first and capture the bound ink **variable IDs** —
     `getStyledTextSegments(['fills'])` → `fills[0].boundVariables.color`. Do this *before* any
     write; the bindings are your only record of the roles.
  2. `await setTextStyleIdAsync(...)`, then read the node's now-resolved `fontName` and
     `loadFontAsync` it. The style changes the font, so loading the *old* font is not enough.
  3. Build the new string from named parts (`lead + mid + link + tail`) and compute offsets from
     their lengths — never hardcode character indices.
  4. Re-bind per range: `figma.variables.setBoundVariableForPaint({type:'SOLID',color:{r:0,g:0,b:0}}, 'color', v)`
     returns a **new** paint — capture it and pass it to `setRangeFills(start, end, [paint])`.
     Then `setRangeTextDecoration(start, end, 'UNDERLINE')` for the link.
  5. Verify by re-reading the segments and asserting each range's bound variable name, not just
     that the text looks right.
- **Editing one line of a two-line mixed-weight label** (e.g. `#EEEBFF\n100`, hex regular + shade
  bold): setting `characters` flattens both to one weight. Instead append then delete —
  `t.insertCharacters(t.characters.length, next, 'BEFORE')` inherits the bold last-char style,
  then `t.deleteCharacters(start, start + oldLen)`.
- **Brand-simulation rows can use a different font per brand.** A loop that loads only the first
  will throw partway through. Load the full set up front, catch per node, and re-run the
  failures.
- A caught exception **does not roll back** earlier successful mutations in the same script
  (`use_figma` atomicity only covers uncaught errors). If you catch, you own the half-applied
  state — report it and finish the job.

## 3. Spacing & tokens

- Every auto-layout `itemSpacing` and padding value must be bound to a spacing token. **Gap and
  padding may be separate scales that resolve differently at the same tier name** — check
  `design_foundations` § Spacing for the values and the traps. Bind gap tokens to gap, padding
  tokens to padding; never cross them.
- **Skip zero values.** Binding a "none" token to every unpadded frame touches nearly every node
  for no theming benefit. Say you skipped them.
- **Off-scale values are the "spacing looks off" bug.** Snap to the nearest tier when within
  ~3px; anything further off is probably deliberate (a 468px gap pushing a header apart) —
  report, don't snap. Log the change distribution (`14→12 ×84`) so the visual delta is
  reviewable.
- Don't bind inside instances — fix the main and let it propagate.

## 4. Use real components, not lookalikes

Hand-built frames that imitate a component (`btn Primary`, `chip success`, `input-sample`) should
become real instances from the component library (config § Figma → `component_library`). Before
swapping:

1. **Check the token bindings on the hand-built version.** If its fills already bind to the
   component's own tokens, the swap is visually safe. If it's raw hex, the swap will change its
   appearance — flag that.
2. **Probe the real component's variants and size first** — build a throwaway instance grid
   off-canvas, screenshot it, pick the variant, then delete the probe. Cheaper than guessing
   variant names.
3. **Respect the real component's minimum size.** A form field whose label has no visibility
   toggle may be over 100px tall; it cannot stand in for a 33px bare box. When the real
   component genuinely doesn't fit the mock's purpose, **leave the mock and explain why** — don't
   wreck the layout to satisfy the rule.
4. Preserve position and order: capture the old node's parent, child index and x/y;
   `insertChild` at the same index; re-apply x/y only if the parent is not auto-layout; then
   remove the old node.
5. Figma cannot embed an instance inside a text run — an inline "text link" inside a paragraph
   stays styled text. Say so rather than leaving it looking unfinished.

## 5. Icons — never a text glyph

**A text character is never an icon.** `✓ ⚠ → ← ↺ ●` typed into a TEXT node must be replaced with
a real instance from the library's icon set. Glyphs don't take the sizing tokens, don't colour
from ink tokens, don't swap, and render differently per font.

The icon anatomy, sizing scale, default variant and reference specimen are in
`design_foundations` § Effects and icons. Three things live on three different layers, and
getting them on the wrong layer is the usual mistake:

| What | Which layer | How |
|---|---|---|
| **Size** | the icon frame / instance itself | bind `width` (and `height`) to a sizing token — **never a raw px value** |
| **Colour** | the artwork layer *inside* the icon | bind `fills` to an ink token via `setBoundVariableForPaint` |
| **Which icon** | the host component's `INSTANCE_SWAP` prop | swap, with the icon library as preferred values |

Match the size to the text it sits beside — replacing a 12px glyph with a 12px sizing token keeps
the row height unchanged. If the icon sets have a style axis (outline weight, filled), set the
default named in § Effects and icons explicitly; sets don't always ship defaulting to it.

Pick by the description on the set, not by the name alone. Search the component library
specifically (`includeLibraryKeys`), or third-party kits in the file drown out the results.

**Ink-token trap.** Ink designed to sit **on a coloured fill** is often near-white
in its bold variants — correct on an amber chip, invisible on a white panel. Warning icons that
inherited an on-fill ink have vanished this way. When an icon sits on the page background, use
ink meant for the background. Always resolve the hex and sanity-check it against what it sits on.

## 6. Placeholder-inheritance trap

A local component used as a generic swatch or tile carries **placeholder content** (e.g. a
swatch's shade caption reads `100`). Every instance that never overrode it silently displays that
placeholder — which is how 17 role-value swatches once all read "100" while pointing at entirely
different tokens.

When auditing repeated data-bearing instances: **derive the correct value from an adjacent
authoritative node** (the token name printed beside it) rather than trusting what's rendered, and
check whether each instance has a real override or is inheriting. Fixing the main only helps the
un-overridden ones; overridden instances must each be edited.

## 7. Lint pass (repeatable, in this order)

Run read-only first, report, then fix. Each step is independently verifiable:

1. **Scope** — list screens; identify driving main components; identify and exclude imported
   reference frames.
2. **Structure** — for a "match my reference" request, diff sibling nodes against the reference
   and list every differing property.
3. **Components** — find hand-built lookalikes; classify swap / leave-with-reason (§ 4).
4. **Icons** — find text glyphs used as icons; replace with icon instances, sizing token on the
   frame, ink on the artwork layer (§ 5).
5. **Text styles** — count styled vs unstyled; group unstyled by size + font + container;
   classify chrome vs specimen (§ 1); apply by role.
6. **Case** — find baked-caps content; auto-fix where the style already transforms, report the
   rest (§ 2.1).
7. **Spacing** — audit gap/padding: already-bound / on-scale-unbound / off-scale; bind, snap,
   report skips (§ 3).
8. **Data integrity** — sequences, numbering and labels that should agree with an adjacent token
   name (§ 6).
9. **Verify** — `screenshot()` every touched region. Styling changes reflow layouts: widening a
   label column, growing a button from 33→40px, or a caption that no longer fits its fixed-width
   column are all real regressions this catches. Fix overflow before reporting done.

## 8. Reporting

Follow config § Writing → `report_style`: concerns first, lists over prose. Always state:

- What changed, with counts and the before→after distribution for anything that moved pixels.
- **What you deliberately did NOT change, and why** — this is the most valuable part. Specimens
  left raw, components that don't fit, mono text with no style available, deliberate gaps,
  decisions that need the person's call.
- Data defects found in passing (duplicate values, invisible swatches, labels disagreeing with
  their tokens) — report even when out of scope; don't invent the correct value.
- Publish and propagation state: which mains were edited and therefore which instances updated.
  A library change reaches other files only after config § Roles → `publisher` publishes it.
