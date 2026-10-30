# Design foundations

The facts the design skills (`ds-design-component`, `ds-token-variables`, `ds-design-journey`)
rely on. The skills carry the workflows.

- **Part 1, Your system** is a fill-in template. `ds-setup` drafts it from your Figma files and
  asks you for the intent; replace each guidance block with your system's facts. The Acme
  examples show the shape of an answer; delete them once yours are in.
- **Part 2, Working in Figma** is general Figma and Figma API knowledge for any system. Keep it,
  and add to it.

Team facts (people, repo, tracker, page conventions) live in `.claude/design-system.config.md`,
not here.

---

# Part 1 · Your system

## Files

Every Figma file the design skills touch: its role, name, key (from the file URL), and what it
holds. Roles the skills look for: **primitives** (raw values), **brand** (brand-varying values,
if separate), **semantic** (what components bind), **component library**, and **utilities** (doc
components, if separate). Merge roles if one file does several jobs.

*Example (Acme):*

| Role | File | Key | Holds |
|---|---|---|---|
| primitives | Acme Primitives | `aBc123…` | Raw colour ramps, spacing, radii |
| semantic | Acme Tokens | `dEf456…` | Semantic roles: background, ink, border, spacing |
| component library | Acme Components | `gHi789…` | The components themselves |

## Token tiers

The tiers, in aliasing order, which collection each lives in, and **which tier components
bind**, in one line. Components binding a lower tier directly is the most common architecture
defect.

*Example (Acme):* Primitives → Semantic, by aliasing. Components bind **Semantic** only.
Primitives are never bound directly.

## Mode axes

Each independent axis a frame can switch: its collection, its modes, the default, and what the
axis means. Typical axes: brand or density, theme (light/dark), breakpoint. Say whether axes
compose independently (each in its own collection) and anything about how one works, e.g. how
dark mode treats already-dark surfaces.

*Example (Acme):*

| Axis | Collection | Modes |
|---|---|---|
| Theme | `semantic-theme` | `light` (default), `dark` |
| Breakpoint | `semantic-responsive` | `desktop`, `tablet`, `mobile` |

How responsive values work, if they do: where the per-breakpoint numbers live, and the single
variable a component binds. **Bind one variable and switch the frame's mode**; never bind three
variables and toggle visibility.

## Component tokens

When a component gets its own tokens rather than binding general semantic ones: the path pattern,
and which tier holds the raw values versus the semantic token. **Prefer a component token where
one exists** over a general semantic one.

Which container backgrounds components bind (a card or container role) versus page backgrounds
(a surface or canvas role): a component bound to the page role is a defect.

*Example (Acme):* `component/{name}/{variant}/{state}/{element}`, e.g.
`component/button/primary/hover/background`. Component dimensions: raw per-breakpoint values in
Primitives (`{component}/{property}/{breakpoint}`), one semantic per property in
`semantic-responsive`. Components bind `background/container/*`, never `background/page/*`.

## Naming

Variable naming rules, the code syntax pattern (what the exporter reads; it may not be the
variable name), and markers that keep a mode or variable out of the code export. The deprecation
pattern is config § Tokens → `deprecated_name`: values are frozen, and nothing new binds to it.

*Example (Acme):* lowercase, `/`-separated. Code syntax `var(--acme-{path})`, and the exporter
reads code syntax, so a rename leaves code on the old path until code syntax is updated too.

## Text styles

The published style groups, and which properties the styles bind to variables. If styles bind
the font family to variables, the **same style renders a different font per brand or theme**:
never pick a font by hand. Record the style that uppercases (via `textCase`), and any known gap,
such as having no monospace style.

*Example (Acme):* `Heading/*`, `Body/*`, `Label/*`. Font family, weight and size bind to
Semantic. `Label/Overline` is uppercase. No mono style: leave mono text raw and flag it.

## Spacing

The gap and padding tokens, their collection and unit. **If gap and padding are separate scales
that share tier names, say so loudly and give both tables.** It's the trap: `small` gap and
`small` padding resolve to different pixels, so binding `itemSpacing` to a padding token (or the
reverse) looks right and is wrong. Mark the tiers where both scales exist with different values.

*Example (Acme):* `spacing/gap/*` and `spacing/padding/*` in `semantic-responsive`, in `rem` at a
16px root.

| Tier | gap | padding |
|---|---|---|
| small | 8 | 12 |
| medium | 16 | 24 |

## Effects and icons

How shadows and other effects are applied (effect styles or variable bindings), where they live,
and how one is chosen (by surface darkness, for instance). Then the icon library:

- **Where icons live** and how they're named.
- **Sizes**: the icon size scale, and the sizing token for each size.
- **Style variants**, if icons come in more than one (outline, filled), and the default.
- **Recolouring**: which layer the colour token binds to, and whether a pre-built icon wrapper
  exists per size.
- **App and logo icons**, if the library has them: their size and sizing token, and one
  reference specimen to copy.
- **Search**: how icons are described so they can be found by meaning.

*Example (Acme):* shadows are effect styles in the component library, `shadow/{low|high}`.
Icons are `icons/*`, sizes 16 / 24 / 32 bound to `size/icon/{sm|md|lg}`, outline by default,
recoloured on the vector inside the instance. App icons are 48px, bound to `size/icon/app`;
`icons/app-acme` is the reference.

## System rules

The idiosyncrasies of *your* system: anything about how it works that changes how a component
is specified, designed or built. Write each as **a rule, and the question it raises for every
new component.** `ds-crd-write` asks these questions in its interview, and `ds-crd-review`
checks drafts against the rules.

Things that often belong here:

- **Brands.** Single brand or several? If several: what may vary per brand, and what must stay
  the same across all of them.
- **Colour families** that only work together, e.g. a status fill, its ink and its border.
- **Customer theming**, white-labelling, or customer-supplied colours.
- **Platforms** the components must work on, and what differs between them.
- **Writing direction, density, motion** or anything else your system treats specially.

*Example (Acme):* Multi-brand. **Status colours never vary by brand**: status fill, ink and
border come as a set from the status tokens. *Question for each coloured part: does it follow
the brand, or is it status?* Customers can't supply colours, so there's no runtime contrast
check.

---

# Part 2 · Working in Figma

## Finding tokens

**Use `search_design_system` against the published library.** The Plugin API
(`getLocalVariablesAsync`, `getVariableByIdAsync`) only sees variables already imported into the
current file. It can't see unpublished or unused library tokens, so it reports a token as missing
when it merely hasn't been imported.

**Don't use an exported token JSON to decide what exists.** It's generated downstream for code,
lags until the library is republished and re-exported, and may be keyed on code syntax rather than
the variable name. For design work, the published library is the authority.

**Spacing, padding, gap and sizing always come from tokens**, never a hardcoded number.

**Reading a resolved value:** `variable.resolveForConsumer(node)`. `valuesByMode` returns the
alias, not a number.

## Publish gates

**Consumers only see PUBLISHED values.** After editing tokens, the source library must be
published before a downstream file's imports, bindings or demos reflect the change.
`importVariableByKeyAsync` fails outright on an unpublished variable, and "not found" almost always
means needs-publish, not wrong key. This is the single most common source of "why hasn't it
changed". Each tier is its own gate: publish them in aliasing order.

Only config § Roles → `publisher` publishes. **Pause and ask at every gate.**

## Building layouts

These apply to everything built in Figma: components, documentation frames, screens and
prototypes.

- **Auto-layout everywhere.** Pages, screens and components are built from auto-layout frames.
  Absolute positioning is rare and deliberate: a badge pinned to a corner, an overlay.
- **Never use a spacer frame**: no empty frame with `layoutGrow = 1`, and no fixed-size empty
  frame to push things apart. A spacer leaks into every instance and into dev handoff, needs
  maintaining across variants, and hides the layout intent. Use the real mechanism:
  `primaryAxisAlignItems`, `FILL` on an element that genuinely fills, absolute positioning with
  constraints, padding, or moving a visibility binding. If no clean mechanism exists, say so and
  ask.
- **Clip content only where it's genuinely needed**: a screen root, a scroll viewport, a frame
  whose content really overflows, or a rounded frame with full-bleed children. Elsewhere, clipping
  hides overflow rather than fixing it: the content is still there, still wrong, still in the
  accessibility tree. It also cuts shadows.
- **Never fake an edge-docked panel with uniform padding.** A side panel or bar that docks to an
  edge is its own frame in the layout, not content pushed inward by padding on the root.
- **Strip the defaults on every frame you create.** `figma.createFrame()` and
  `figma.createAutoLayout()` ship a solid white fill and clip content on. Set `frame.fills = []`
  and `frame.clipsContent = false` immediately. A frame keeps a fill only when it's bound to a
  token.
- **Every gap and padding binds to a token**, from the right scale (§ Spacing).

## Working in Figma via MCP

**Facts** that more than one workflow depends on. **Procedures** (which keys, what order, how to
recover) belong in the skills: a step-by-step added here probably belongs in a skill.

**Any claim here about what the Figma API cannot do is a dated observation about someone else's
product.** Write the date, and re-test before trusting it.

### Reads and writes

- **A rejected or interrupted `use_figma` write may already have landed.** A call reported
  "rejected by user" had in fact fully run (2026-09-25). Re-read the file state before re-running
  anything, say what actually landed, and guard mutations with a verify-it-exists check so a
  re-run fails safely.
- **A script is transactional.** One thrown error rolls back everything, so put read-back
  reporting in a separate call.
- **Reads straight after a mutation are stale within the same script.** `defaultVariant` still
  reported the old variant right after a reorder; a fresh call showed the truth. Verify read-back
  in a separate call, not just on error. *(2026-09-09)*
- **`get_metadata`'s page list includes only LOADED pages**, without saying it's partial. It
  reported 2 pages in a 63-page file. `figma.root.children` inside `use_figma` returns them all,
  and child counts on an unloaded page also read low: load the page before trusting a count.
- **`importComponentSetByKeyAsync` returns the PUBLISHED mirror, not the editable node.** The same
  set exists twice: a local node on its page (`remote: false`, writable) and a library mirror
  (`remote: true`, no parent, read-only) sharing one key. Read published state from the mirror;
  write to the local node, and **check `.remote` before any mutation**. *(2026-09-09)*

### Variables and paints

- **Bind paint colour with `setBoundVariableForPaint`.** `setBoundVariable` doesn't work for
  paints.
- **Imported-but-unused variables get pruned between scripts.** Import and bind in the same
  script, and null-check before binding.
- **A paint binding can die silently between scripts.** The binding stays (`boundVariables`
  shows the alias, `resolveForConsumer` returns the right value) but the paint renders nothing.
  **The tell: `paint.color` reads `{r:1,g:1,b:1}` instead of the resolved colour.** At risk:
  variables not already used elsewhere in the file. Re-assigning the fill or stroke does not fix
  it; **create the node fresh in the script that imports the variable**, or clone a good node.
  Verify by rendering (`get_screenshot`), not by reading state: `absoluteRenderBounds` and an
  in-script `node.screenshot()` both missed it. *(2026-09)*
- **Effects bound through an effect style show the style's internal variables** in
  `effects.boundVariables`, which can look like a forbidden lower-tier binding. **Read
  `node.effectStyleId` before judging an effect.**

### Text

- **Apply the text style; never reproduce it by hand.** Manually matched text carries no style
  binding, so it can't follow brand or theme and goes stale silently.
- **`setTextStyleIdAsync()` changes the node's resolved font.** Before writing any
  font-dependent property afterwards (restoring `textDecoration`, say), `loadFontAsync` the NEW
  font, or it throws `Cannot write to node with unloaded font`.
- **A hugging text node with `maxWidth` wraps, but the box stays at `maxWidth`.** Whatever follows
  it in the row (a required-field asterisk, say) pins to the end of the box, not to the last word.
  *(2026-10)*
- **Switching modes or editing instance text needs every brand's fonts loaded.** On an
  "unloaded font" error, parse the family and style from the message, load it and retry.

### Component sets and auto-layout

- **A set's default variant comes from canvas POSITION, not child order.** The default is
  whichever variant sits top-left. *(2026-09-09)*
- **Sets can be `GRID` auto-layout**, where `x`/`y` are computed and can't be written. Place with
  `child.setGridChildPosition(row, col)`, called on each child, not on the set. The readable anchors are `gridRowAnchorIndex` and
  `gridColumnAnchorIndex`; **`gridRowStart` / `gridColumnStart` throw on read**. The setter
  throws on an occupied cell, so swapping cells in a full grid needs `gridColumnCount` (or
  `gridRowCount`) raised temporarily to park in. *(2026-09-09)*
- **Grid children ignore the cell.** Auto-layout children dropped into a GRID frame hug, and
  `FILL` is rejected. Pin the tracks to fixed sizes and resize the child to the cell.
  *(2026-09-16)*
- **Variants of mixed sizes need uniform FIXED tracks.** With HUG or flex tracks, the grid sizes
  each track to its biggest variant and moves the others as they change. Set every
  `gridColumnSizes` and `gridRowSizes` entry to FIXED, sized to the largest variant.
  *(2026-10)*
- **`resize()` on a FILL or HUG child silently flips it to FIXED.** Restore the sizing mode
  after resizing.
- **`layoutSizingHorizontal` reads FIXED on a hidden node**, even straight after setting it to
  FILL, and the setting may not stick. Set `layoutAlign = 'STRETCH'` while the node is visible,
  then hide it. *(2026-10)*
- **`rescale()` scales text with the artwork**, so it shrinks type too. Prefer `resize()` on
  auto-layout instances; keep `rescale()` for things that must stay proportional.
- **`SPACE_BETWEEN` centres a lone child.** When a boolean prop hides a child, Figma drops it
  from the *instance's* children entirely, so a two-child `SPACE_BETWEEN` row becomes one child,
  centred. The library variant looks fine because the child is visible there. **Fix: bind the
  visibility to the artwork *inside* a fixed-size wrapper**, so the wrapper always holds its
  place. Clear an existing binding with `componentPropertyReferences = {}`; `null` throws.
  *(2026-08-28)*
- **`resize()` doesn't refit a GRID set.** After converting a set to GRID or placing children, the
  frame keeps its old size and squeezes flex tracks. Set `gridColumnSizes` and `gridRowSizes` to
  HUG (FIXED when variants differ in size, above), then `resize()` explicitly to padding×2 + Σ track sizes + gaps.
- **GRID stretches variants to the row height.** Set every variant's `layoutSizingVertical =
  'HUG'` after converting, and again after any `resize()`. Rows beyond `gridRowCount` collapse to
  zero height, so the set must hug its height.
- **Auto-flow grids.** Some GRID sets have `gridAutoTracks = 'ROWS'` (missing from the bundled
  typings, so probe the live node). Symptoms: setting `gridRowCount` throws "cannot set … while
  GridAutoTracks is set", and `setGridChildPosition` throws "automatically positioned items — use
  parent.insertChild()". `gridAutoTracks = 'NONE'` unlocks the counts, but items may stay
  auto-positioned; then cell = layer index, row-major. Set `gridColumnCount` first, sort the
  children into row-major order, and `insertChild(i, child)`.
- **Variant option order can be fixed by renaming.** `editComponentProperty` rejects
  `variantOptions`, and reordering children doesn't change the picker, but renaming each value away
  and back, in the order you want, does. *(2026-10)*
- **Grid geometry keeps reflowing between scripts** after big variant additions, so geometry read
  in the script that added variants is stale. Position labels around a set in a fresh script, and
  verify them in another. Screenshot the bottom of the set, not just the top.
- **A single override on a nested node can't be reset by script.** `resetOverrides()` exists
  only on instances and strips *all* their overrides. Writing the main component's value back
  doesn't clear the override flag and can bake in different behaviour (an auto gap becomes a
  literal `0`). Set what renders correctly, then hand the person the one-click fix: right-click
  the layer → **Reset changes**. *(2026-08-28)*
- **`resetOverrides()` misses some overrides**: values set with `setProperties`, and content
  inside a nested instance's own slot. Set those back to their defaults explicitly. *(2026-10)*
- **Renaming a main component doesn't rename its instances' layers**, or nested instances' names.
  They keep the old name until a script renames them. *(2026-10)*

### Cloning and swapping

- **Cloning a variant within a set strips all `componentPropertyReferences`**: text `characters`,
  `visible` and `mainComponent` swap refs, silently. The set-level property definitions survive, so
  the damage is invisible until a consumer selects the new variant and the props vanish from the
  panel. After every clone batch, copy refs from the source variant by layer path or name (merge
  only what's missing: `{...want, ...current}`), then check ref-count parity per variant. Cloning a
  standalone COMPONENT, or a whole set, preserves refs.
- **Cloning a variant within a set degrades its SLOT nodes to plain FRAMEs** and strips
  `slotContentId` and `visible` refs. Cloning a standalone COMPONENT keeps slots, and
  `combineAsVariants` re-homes the slot property onto the new set. **Recovery:** set
  `componentPropertyReferences = { visible, slotContentId }` on the degraded frame using the set's
  *existing* slot property ids, and Figma re-promotes it to a SLOT. It still reads FRAME in the same
  script; verify in a fresh call. If the slot property doesn't exist on the set yet, one manual
  Convert-to-slot creates it.
- **Cloning within a set copies Dev Mode annotations.** Strip them from the clones so an annotation
  stays on one variant.
- **`swapComponent` keeps instance overrides.** Stale override bindings (old fills, effects) sit on
  top of the new main component and keep the old look. Call `resetOverrides()` after a swap where
  the target's own design should win, then reapply deliberate overrides.
- **Text with per-range fills** (`boundVariables.textRangeFills`): assigning `node.fills` doesn't
  clear the range binding. Load the fonts, then `setRangeFills(0, characters.length, [paint])`.
- **`setBoundVariableForPaint(paint, 'color', null)` silently unbinds** when a pruned variable
  resolves to null: a raw fill, no error. Null-check every resolved variable and throw if missing.

### Slots

- **SLOT properties can't be set with `setProperties`.** It throws `Slot component property
  values cannot be edited`. The slot is a real child node (`type: 'SLOT'`) that accepts
  `appendChild`: add content structurally. *(2026-09-16)*
- **A variant change on one slot child can invalidate sibling node references** for the rest of
  the script, even with fresh lookups. Configure instances before appending them, or split the
  work across calls.
- **A hidden child disappears from an instance's `.children`.** Never iterate by a captured
  index; re-query after each mutation.
- **`FILL` on slot children must be set in a later call** than the slot surgery.

### Annotations and descriptions

- **Annotations are writable via MCP**: `node.annotations = [{ labelMarkdown }]` *(2026-09-09)*.
  They surface in **Dev Mode**, so they're for developers and dev agents.
- **Notes a designer needs** at insert time go in the component set description instead. It's
  also how a note reaches a dev agent: `get_design_context` returns the description (and dev
  resource links, below), not Dev Mode annotations.
- **A keyframe's easing controls the segment going *into* it**, so the first keyframe's easing is
  never used. Animations go on an instance in the motion frame, never on the set
  (`ds-design-component` → `documentation-frames.md` § Motion frame). *(2026-10)*
- **An agent can't post Figma comments.** Draft the comment, with the node link, for a person to
  paste. *(2026-10)*
- **Dev resource links can't be written by an agent.** `use_figma` blocks them both ways, so a
  person adds them. An agent reads them via `get_design_context` on a variant, under
  **Documentation**; nested components' links don't come through. *(2026-10-06)*

## Where to go next

| Task | Skill |
|---|---|
| Building or extending a library component | `ds-design-component` |
| Editing variables in the token files themselves | `ds-token-variables` |
| Screens, journeys, prototypes and their lint | `ds-design-journey` |
| Normalising app or logo SVG assets | `ds-asset-normalize` |
| Updating a CRD after design or build | `ds-crd-update` |
