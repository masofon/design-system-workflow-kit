# Documentation frames

Reference for `ds-design-component` § 1. A component page carries up to four top-level frames, left
to right, config § Figma → `frame_spacing` apart, in the order config `doc_frames` gives:
typically **component → anatomy → usage → modes**. They're split because components get big, and
each frame answers one question.

**Every name, size and colour here comes from config § Figma → Library conventions** and the
team's own design system. This file sets the structure; the config and the DS set the look.

**Which frames a component needs is a judgement.** Anatomy mostly for multi-part components;
modes only where the tokens really vary, since a mode that renders identically teaches nothing.

## Kit layout or team layout

Config § Figma → `page_layout` decides which layout a page follows.

- **`team`**: the team kept its own component-page layout at setup. **Copy config
  `reference_page`** and follow the Library conventions keys, which record the team's frames,
  order, naming, headings, fills and spacing. The frame-by-frame guidance below applies only where
  it doesn't conflict, and as a checklist of what each frame should show. Where theirs has no
  equivalent of a kit frame, don't add one; mention the gap in your report.
- **`kit`**: everything below applies. Match config `reference_page`, the most recent page
  carrying the ready-for-dev symbol (config § Figma → Page status symbols), copying its structure
  rather than starting from scratch. On a new library with no such page yet, build from this file
  and the first finished page becomes the reference.

**Either way, the general rules hold**: text styles on every text node, tokens for every colour,
gap and padding, no spacer frames, annotate once, live instances only.

## Building blocks

**If config `doc_utilities` names the library's documentation components, build every frame from
them**: place instances, don't draw these by hand. The parts every frame needs:

| Part | Use |
|---|---|
| Section header | Every frame's header: page heading plus a one-line description |
| Doc row | One row: a doc-note beside a specimen slot. Drop live instances into the slot |
| Doc note | CRD reference eyebrow, title, description. Inside a doc row, or on its own beside the set |
| Doc subhead | Groups rows within a frame, e.g. Usage's two halves |
| Component label | Names the specimen: a demo instance, or the main component or set itself |
| Anatomy pin | Numbered anatomy pin |
| Axis label | Row and column labels around the variant matrix |

**If `doc_utilities` is `—` or empty, build them once, from the team's own design system**, as
local components, and reuse them on every page so every page reads the same:

- **Text** uses the team's text styles (`design_foundations` § Text styles): an eyebrow or
  overline style for the CRD reference, a heading style for titles, a body style for captions.
  Never a hand-picked font, size or weight.
- **Colour** binds the team's tokens: the page or container background for fills, their ink
  tokens for text, their border tokens for dividers and leaders. Never raw hex.
- **Spacing and radius** bind the team's gap, padding and radius tokens (`design_foundations`
  § Spacing; config `frame_radius`).
- **Icons** are instances from the team's icon library.
- **Propose the set first.** List the parts, the style or token each one uses, and where they'll
  live (a utilities page or file). Build only on the designer's yes, then ask the person to record
  the result in config `doc_utilities`.

**Doc row text:**

- **CRD reference** (eyebrow): where the rule lives, e.g. `CRD § Behaviour · B3, B4, B5` or
  `CRD § 4 Usage · Patterns`.
- **Title**: what the row shows.
- **Description**: a one-line caption of the specimen, e.g. "Five avatars and three hidden, as the
  container narrows." It never re-explains the rule; the eyebrow points at it. *Using it in Figma*
  rows are the exception: they give the instruction, briefly, because that's where it gets read.
- **Write it the way a person would say it.** Short, plain sentences a designer reads at a glance,
  not spec shorthand: "Hides the label when there's no room", not "label: hidden @ <min-w". No
  statement IDs, token paths or arrows in the title or description; the eyebrow carries the
  reference.

## Shared base

- Frames are named per config `frame_names`.
- Vertical auto-layout. Fill bound to config `frame_fill`, radius `frame_radius`, no clipping, gap
  and padding bound to `frame_spacing_tokens`, each from the right scale (`design_foundations`
  § Spacing).
- No bordered tiles or cell grids: content is doc rows.
- **Header:** a **fresh** section header instance, never one cloned from another frame (a clone
  lost its description). Page heading per config `frame_heading`, numbered in frame order. The
  description is one line about what *this frame* shows, not about the component.
- Text uses text styles and colours use tokens, no exceptions. Text content follows config
  § Writing → `text_case`. Icons are icon instances, never typed glyphs.

## Component frame

The header, then a row: a notes column beside the matrix.

- **Notes:** doc notes, each with its CRD reference.
  - One on the variants: the count, and which axes are Figma-only ("48 variants: breakpoint ×
    size × variant. Breakpoint is Figma-only.").
  - One per code-only prop, saying what to do in Figma instead ("No Figma property: edit the label
    text instead.").
- **Matrix:** a plain frame holding the set, its backdrop tiles and its axis labels.
  - **Grid**: the set's layout follows `SKILL.md` § 3, Laying out the set, which also covers the
    sizing traps.
  - **Axis labels**: one on every row and every column, outside the tile outset, above the tiles in
    layer order.
  - **Backdrop tiles** *(only where a variant needs a background to be seen, e.g. a transparent
    component)*: one rectangle per variant, named `backdrop / {variant} / {size}`. Radius
    `frame_radius`.
    - **Fill** bound to the page background token by default, or the container background where
      the component needs it: the designer's call.
    - **Outset** config `specimen_tile_outset` beyond the variant's bounds, so a variant with no
      padding still sits inside its tile.
    - **At least** `specimen_tile_gap` between tiles, so the set's grid gap is at least tile gap +
      2 × outset.
    - A set can only contain variants, so the tiles sit in the matrix, behind the set.
    - Position them from fresh variant geometry in matrix coordinates (set position + variant
      position), in a later script than the one that built the grid.
- **Annotation**: one Dev Mode annotation on the set (not the frame) saying any tiles are demo-only
  and not part of the component. The same note goes in the component description.

## Anatomy frame

The header, an overview, then one doc row per part.

- **Overview**: one instance, at its largest size, in the default variant.
  - Anatomy pins, each with a 1px dashed leader in an ink token to the part's edge, alternating left
    and right. For text, measure the real glyph width and end the leader there.
  - A transparent container gets a dashed 1px outline, named as demo-only.
  - **Legend**: numbered lines of the CRD's Anatomy nouns, verbatim. Nouns only; the rows say the
    rest.
  - **Footer**: other sizes share the anatomy, and the outline is a demo marker.
- **Rows**, in legend order: eyebrow `CRD § Anatomy · {IDs}`, title = the noun, caption = the
  configuration shown ("X-large, label off: the ring between avatars."). Where the part is
  another component, the component label names it.

## Usage frame

The header, then two doc-subhead groups, in this order.

1. **Using it in Figma**: what trips people up, such as slots, Figma-only axes, or a nested
   property to leave alone. Intro: read these before building a screen.
2. **Examples**: the default first; each optional part off; each option of a consumed component; a
   swapped icon with different copy; the behaviours; the smaller size inside a real host component
   (a card, say), not a bare frame.

- **Live instances** with properties changed. Nothing detached.
- **Real product copy**, and icons chosen by description. Never placeholder text ("Heading text").
- **A behaviour sequence goes step by step in one row.** Overflow B3 → B5 is one row, not three.
- **Each caption says what differs from the default.**

## Modes frame

The header, then one doc row per mode axis that matters for this component
(`design_foundations` § Mode axes): typically theme, breakpoint, and brand or density.

- **Cell**: a wrapper frame using `setExplicitVariableModeForCollection`. Get the collection from
  an imported variable's `variableCollectionId`, and find modes by name. Bind the wrapper's fill to
  the page background token so it re-resolves under the mode. The label sits outside the wrapper.
- **Theme**: the component in each theme mode, side by side.
- **Responsive**: one cell per breakpoint at config `responsive_widths`, the instance filling the
  wrapper. A variant that sets its own breakpoint ignores the wrapper's mode: switch the instance's
  breakpoint property instead.
- **Brand and density**: `resolveForConsumer` first. Show a brand only where it differs from the
  default brand (`design_foundations` § Mode axes). Brands that resolve identically for this
  component's tokens are shown once, and the caption says so ("Compact matches Brand B, so it isn't
  shown.").
- **Fonts**: other brands may use unloaded fonts (`design_foundations` § Working in Figma via
  MCP, Text).
- **Verify with `resolveForConsumer`.** Screenshots of different modes can look alike.

## Gotchas for all four

- New frames ship with a white fill and clipping on. Strip both at creation (`SKILL.md` § 5).
- `resize()` turns hug into fixed: set hug again after resizing. Instances inside wrappers need
  vertical hug too.
- Build each frame in one script, and check its geometry in a fresh one.
- **No spacer frames** (`SKILL.md` § 5).
- **Annotate once.** A note that applies to every variant goes on one variant, with the context on
  the page beside it.
