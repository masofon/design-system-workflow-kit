---
name: ds-design-component
description: Design-side workflow for creating or extending components in the design system's Figma component library, core or stable — establishing whether the person is a designer and whether the component is stable or core, choosing an entry path from existing design, concept flow, shipped code or CRD alone, then page setup and context cards, variant axes, token creation and binding, slots and props, verification, publish gates and documentation. Use when building or modifying Figma components, setting up a component page, adding size or variant axes, wiring component tokens, building a stable component in Figma, or promoting a stable component into the core library.
---

# Design component (Figma component library)

Building components in the **component library** (config § Figma → `component_library`). Read
the foundations doc (config § Paths → `design_foundations`) first: Part 1 for this system's files,
tiers, modes, text styles and spacing; Part 2 for the Figma API facts this skill relies on.

## The file is edited live by others

**Any change you didn't make was deliberate.** Nodes will move, vanish or get renamed between your
scripts. That's collaboration, not corruption.

- Don't investigate, restore or recreate something that vanished. Adapt, and say so in one line.
- Flag it only if it breaks something that must hold: a token binding, a property reference, a
  published instance.
- Re-read live state before acting on geometry, and don't cache node ids across a long session.
- **Odd geometry may be deliberate.** An offset or overlap you didn't expect may be an authored
  state. Ask before "fixing" it.

**Never edit the skills or their reference docs from here.** That means config § Paths →
`skills_dir`, `crd_how_to`, `crd_template` and `design_foundations`. They belong to config
§ Roles → `skill_owners` and change in their own PRs; an edit made inside a component PR collides
with theirs and slips past review. If a rule looks missing or wrong, say so, propose the wording,
and ask the person to raise it with the skill owners.

## Check the model

If this session isn't running on the most capable model available, say so once and recommend
switching before starting. Design calls made here become the component every consumer gets, and
contributors often can't judge the advice themselves. Recommend, never block.

## Two questions before anything else

**Is the person a designer?** Designers get options and loops, not tutorials. Non-designers get the
reasoning behind a choice, industry defaults, and no assumed Figma fluency.

**Is this core or stable?** *(Only if config § CRDs → `stable_components` is Yes; otherwise
everything is core.)* Core explores the options and builds the matrix out. Stable moves: meet the
CRD's requirements, make it usable, stop.

**Stable relaxes exploration, never construction.** Correct tokens from the binding tier,
correct naming and structure are identical either way. A stable component lacks design
consideration, not build quality. Built with raw values, it's a deferred rebuild plus a CRD that
lies about what exists.

## Entry path

They differ in what counts as the truth:

1. **A component design exists**: the design is the truth, the CRD governs behaviour.
2. **A concept or flow design exists**: promote the instance into a real component first.
   **The concept's raw values are intent, not spec.** Before building, map each one to a token and
   show the person a table: value → token → gap (what's off, or "no match"). **Where nothing
   matches, ask.** Never round to the nearest token or invent one. Gaps that need a new token join
   § 0's list. **Defaults get neutral copy** that says what goes there ("Card title"), each
   exposed as a TEXT prop. The flow's copy stays as overrides on the concept instance and moves
   into the usage examples (`documentation-frames.md` § Usage frame).
3. **Code exists, no design**: the **code is authoritative**. Deviations are findings to report,
   not silent improvements.
4. **Neither**: **nothing is authoritative**. The output is a proposal. Say so, and flag every
   decision you had to invent.

**Never triggered by a CRD changing.** Always started by a person.

## Check you can write, before you write anything

Run `whoami`: it returns seat type per organisation, and prompts sign-in if needed. An editing
seat in config § Figma → `org` proceeds. **A view-only seat,
or no access to that org, stops before anything is created.** Never discover this through a
failed write mid-build: a half-built component in the library is worse than none.

**If you can't write,** hand over what a designer with access needs: entry path and truth source,
anatomy, the variant axes the CRD describes, which tokens need creating (and so
which publish gates apply), and anything undecided, phrased as a question. Present it in chat so
they can carry on building.

**Then raise a design request.** Hand to `ds-ticket` § Design request, which files an unassigned
work item in the backlog for the design system team, linking the CRD and the stable build. It
works without a clone of the design system repo, which is normal here: the person is in their own
repo. Confirm before filing, and hand back the item URL.

**Filing it is the point, not a formality.** The design system team needs to know a stable
component exists: it affects what they plan, and it will eventually need hardening. A stable
component nobody filed is one they find later, in someone's screens.

## Discovery: three checks

**1. Does a CRD exist?** Look in config § Paths → `crd_dir`; `ds-crd-read` has the name-matching
rules. **A new build requires one.** If there isn't one, hand to `ds-crd-write`. (New builds only.
Implementing or reviewing an existing component with no CRD isn't blocked.)

**2. Does the system already have it?** Check config § Repo → `component_code_dir` and the
component library. Use it, or extend it: a second overlapping component is worse than an awkward
prop.

**3. Does a stable component already exist?** Look for a set named per config § Figma →
`stable_set_name` or `wip_set_name`, and `lifecycle: stable` in the CRD. **If one exists, you're
hardening, not creating.** Missing it means building a parallel component, which orphans every
instance of the stable one and forces a retrofit on every consumer.

## Stable components

*Only if config § CRDs → `stable_components` is Yes.*

- **Properly designed**: the set is named per `stable_set_name`, and the page is sorted into its
  category with the core pages (§ 1). The set name says it's stable.
- **Figma still unfinished**: it's WIP whatever the code state. Name it per `wip_set_name`, on one
  of config `wip_pages`.
- The **CRD lives in the repo like any other**. `ds-crd-update` sets `lifecycle: stable` and the
  `stable:` block. The **stable code doesn't go into the core library.**

**Hardening promotes the existing set**: drop the stable or WIP segment from its name, move the
page into its category (§ 1), raise the bar. Figma links instances by key, so the rename doesn't
break them. Promotion is config § Roles → `promotion_gate`'s call: propose it, don't do it unasked.

**Promotion doesn't earn the ready-for-dev symbol, and you never add it.** Promotion is where the
page lives; the symbol (config § Figma → Page status symbols → `ready_for_dev`) asserts the
component is ready for dev. That's a human judgement needing the design review only a person can
hold (`crd_how_to` § The ready-for-dev gate). Leave the page without it and say what's
outstanding. A stable component is held to the same build bar: **stable relaxes exploration,
never construction.**

## 0. Before building

- Read the CRD (`ds-crd-read` locates it; `ds-crd-review` critiques it).
- Read `documentation-frames.md` beside this skill. It sets the frames the page needs.
- Identify tokens that don't exist yet. Token creation comes FIRST, because of the publish gates.
- Inspect the nearest sibling component before creating anything: variant naming, layer naming,
  prop conventions, bindings. Match, don't invent.
- **Changing an existing set? Audit it first.** A read-only pass, then a report of what you'd
  change, before any edit. The person sees the blast radius before it happens.
- **Check this component's CRD front matter first, and offer the fix straight away** (via
  `ds-crd-update`): designing it means `lifecycle` should be `active`; a finished design review
  means `design_specs: complete`. Stale front matter misleads every other skill and person.
- **Set `lifecycle: active` when a designer starts core work**, via `ds-crd-update`, never by
  hand. At the start, not the end: `active` is how anyone else can tell it's being worked on now.

## 1. Page setup

Page conventions are config § Figma → Library conventions and Page status symbols. Read them first.
**If `page_layout` is `team`, copy `reference_page`** and follow the team's recorded conventions;
the kit's frame guidance (`documentation-frames.md` § Kit layout or team layout) fills only the
gaps. The rules below on naming, symbols, placement and descriptions apply either way.

- **Placement**: one of `wip_pages` while WIP. Core and stable pages sit under their category
  header, from the CRD's `category:` front matter (values in config § Paths → `classification`).
  No `category:` yet? Get it added via `ds-crd-update` first. Category headers follow the
  classification doc's order, and pages sit **alphabetically** under their header, ignoring any
  prefix glyph and the status symbols. A category with no header in the library: ask where the
  page goes.
- **Page naming**: per `page_name`. The symbols stack. You set `in_progress` while work is under
  way, and `amendment_in_design` while an amendment is designed but not shipped (remove it when
  it ships). **`ready_for_dev` is not yours to set**: only config § Roles →
  `ready_for_dev_awarder` awards it. Replace any older text suffixes (`- WIP`, `- Deprecated`)
  with the matching symbol when you next touch a page.
- **Leave `protected_pages` alone.**
- **Canvas background**: `canvas_background`, if set. Check it on every page you touch, not only
  at setup: pages drift.
- **Context card** (`context_card`) **carries page metadata only.** Anything said elsewhere
  (variants, usage, accessibility, integration) isn't repeated on it; variants are described
  beside the set (`documentation-frames.md`). Copy the card from `reference_page`.
  **If the library has no context card yet**, build one once, the same way as the doc utilities
  (`documentation-frames.md` § Building blocks): from the team's own text styles, tokens and
  components, proposed to the designer before it's built, then recorded in config `context_card`
  via the person.
  - **Header**: a link to the tracker container (config § Tracker → `item_url`) and to the CRD,
    plus contacts and related components.
  - **Notes**: **Description** (from the CRD's Purpose), **Status** (from the `status:` front
    matter; update it whenever `status:` changes), and **Changelog** last, each entry a date line
    then a one-line description. The first entry cites the CRD version built from.
  - **Older cards**: when you touch one, remove sections that duplicate the CRD or the doc frames,
    and fix the header links.
- **Dev resource link**: the CRD's URL on the component set. It surfaces in Dev Mode for people,
  and for a dev agent via `get_design_context` on a variant. **An agent can't add one**
  (`design_foundations` § Working in Figma via MCP): report the exact URL for the person to add.
  **Always the integration-branch URL**, even before the CRD's PR merges: a PR-branch URL breaks
  when the branch is deleted.
- **Component description is on the set only.** Variants carry none, or `get_design_context` hands
  dev both and the variant copy goes stale first. **Three things at most**: a one-line purpose, a
  note that any backdrop tiles are demo-only, and the CRD link. It reaches the dev agent in full,
  so anything else (anatomy, props, rules) restates the CRD and drifts from it.
- **Component frame**: a top-level frame named per `frame_names` holds the set, with demos
  alongside. Card and component frame sit side by side, `frame_spacing` apart, matching sibling
  pages.
- **Documentation frames** (anatomy, usage and modes, beside the component frame) follow
  `documentation-frames.md`.
- **One home per fact.** The CRD says what must be true. Doc-row captions point at it and describe
  the specimen. Annotations carry pointers and Figma-only facts; how a literal is calculated (an
  overlap formula) lives in its annotation, nowhere else.
- **Changelog entries are ONE dated line**: what changed, and the version or PR. The bar: would a
  designer opening this page next month be surprised by the component without it? A new variant
  axis, a token rebind, a breaking rename, a deprecation: yes. Defects fixed while building,
  rejected options, debugging: no. Report
  those in chat, and put a Figma API gotcha worth keeping in `design_foundations` Part 2 (via the
  skill owners), not on a card. The card records the component's current state, not the story of
  its build.

## 2. Tokens: component dimensions

The pattern is `design_foundations` § Component tokens. In general:

1. **Raw values** go in the tier that holds them, one per property, variant and breakpoint, with
   scopes copied from siblings.
2. **One semantic variable** per property and variant in the binding tier, its breakpoint modes
   aliasing the raw values 1:1, and its code syntax set per `design_foundations` § Naming. No
   hardcoded mode values.
3. **Publish gates** (`design_foundations` § Publish gates): each tier must be published before
   the next can import its keys, and the binding tier before the library can bind.
   `importVariableByKeyAsync` throwing "not found" almost always means needs-publish, not wrong
   key. **Only config § Roles → `publisher` can publish: pause and ask at each gate.**
4. Colour tokens follow the theme axis instead (`design_foundations` § Mode axes). Creating or
   changing a token's VALUE is `ds-token-variables` work, with its own contrast checks.

## 3. Variant axes & component sets

- **Set names follow config § Figma → `set_name`**, lowercase kebab-case: `actions/button`,
  `forms/search-bar`. Feature-specific components go in their category, never a feature folder.
  Maturity is the only other folder: `stable_set_name` and `wip_set_name` (the dot keeps WIP
  unpublished). Parts are `{category}/{name}-{part}` (`navigation/stepper-step`). Hidden parts
  keep a leading dot on the name, `{category}/.{name}-{part}`, and stay unpublished. Deprecated
  sets follow config § Tokens → `deprecated_name`'s pattern. `icons/*` is exempt.
- **Axes come from the CRD, not convention.** `breakpoint × size × variant` is a common shape,
  not a rule.
- Variant props derive from component NAMES. Rename variants (`breakpoint=desktop, size=small`);
  add an axis by renaming all of them and cloning.
- **Sparse matrices are fine.**
- **Check every literal at every breakpoint.** If what a literal depends on changes by breakpoint
  (an overlap that tracks a responsive size), add a **Figma-only `breakpoint` axis**. Figma-only
  axes and props are the reverse of code-only ones: a Dev Mode annotation saying "don't build", a
  doc-note beside the set, and a "Figma-only" row in the CRD's Properties table via
  `ds-crd-update`. Expect it rarely.
- **Auto-layout set traps:**
  - Sets are often VERTICAL auto-layout with FILL variants. Appending or cloning reflows
    everything.
  - `resize()` flips FILL to FIXED (`design_foundations` § Working in Figma via MCP): restore it.
  - Fluid-width pattern: variant root FILL plus a bound `maxWidth` (and `minWidth`). Instances
    then fill up to the cap, matching `width:100%; max-width: var(...)`.
- `setBoundVariable('maxWidth'|'minWidth', v)` works on auto-layout roots, after the publish
  gate.

### Laying out the set

The Figma API facts behind these are `design_foundations` § Working in Figma via MCP
(Component sets and auto-layout, Cloning and swapping).

- **Rows and columns.** Rows = the axis designers scan down (often size or breakpoint);
  columns = the remaining functional combinations, in the set's existing axis order. Match the
  nearest sibling set if it already has a pattern.
- **GRID auto-layout over absolute positioning.** Convert stacked VERTICAL sets to GRID
  (`layoutMode='GRID'`, row and column counts), with a gap of at least config § Figma →
  `specimen_tile_gap`, or tile gap + 2 × `specimen_tile_outset` when the component frame has
  backdrop tiles (`documentation-frames.md`). Place children with `setGridChildPosition(row, col)`.
- **Park, then place.** `setGridChildPosition` throws on an occupied cell, so a bulk rearrange
  needs two phases: temporarily double `gridRowCount`, park every child at `(targetRow + rows,
  targetCol)`, move each to its target, then shrink the count back.
- **Canvas position sets the default variant**: put the default top-left.
- **Absolute-positioned sets** (legacy): even padding on all sides and gutters of
  `specimen_tile_gap`; column pitch = widest variant + gutter, row pitch = tallest + gutter.
  Resize the set to fit.
- **Big sets** (more than two functional axes): a block per value of the outermost axis, each
  repeating the first block's internal arrangement, stacked vertically.
- Never leave variants stacked or overlapping. **Never reorder variant names or axes to achieve a
  layout**; change position only.
- **Re-layout invalidates labels on the page.** Row and column labels around the set describe the
  old arrangement: rewrite or reposition them, in a fresh script, and verify in another.
- **Cloning variants** strips property references and degrades slots: after every clone batch,
  restore refs from the source variant and check ref-count and slot-count parity.

## 4. Props & slots

- Visibility booleans: `Show {thing}` via `componentPropertyReferences.visible`. Check for
  **orphaned props** after restructures; rebind or delete them.
- **Bind visibility to the artwork, not its wrapper, in a `SPACE_BETWEEN` row.** A hidden child
  drops out of the instance entirely, and a lone child under `SPACE_BETWEEN` centres
  (`design_foundations` § Working in Figma via MCP). A fixed-size wrapper that always stays
  visible keeps its place. Declare the side effect: the row no longer collapses when the child is
  off.
- **Code-only props**: a prop code has that Figma can't show. To keep it in the properties panel,
  nest an exposed instance carrying it inside an absolutely positioned, clipped 0.01px frame. The
  prop stays exposed, and its value survives size, variant and alignment switches. Where it can't
  be exposed, say what to do in Figma instead. Either way: a doc-note beside the set, **one** Dev
  Mode annotation on one variant (§ 8), and a flag that the code-to-design mapping must change.

### Icons

- **Icons are always an icon component instance**, never a character typed into text (`✓`, `⚠`,
  `←`, `×`). A glyph doesn't follow the icon set's style, doesn't scale or align like an icon,
  renders differently per font, and is invisible to the token system. Audit text nodes with
  `/[✓✔✕✖✗✘✅❌⚠ℹⓘ★☆→←↑↓↔⋮▸▾]/`; `×` in dimensions ("12×12") and `•`/`·` separators are
  legitimate typography. To replace a glyph, set the component's own icon prop if it has one;
  otherwise use a horizontal auto-layout row of icon instance plus text, and copy the glyph's
  fills onto the icon's vector so its colour token carries over.
- **Use the icon wrapper if the system has one** (`design_foundations` § Effects and icons): a
  pre-built frame per size. Copy the size you need; don't resize another.
- **Bind the colour token where `design_foundations` says**, typically on the layer inside the
  wrapper, with `setBoundVariableForPaint`. A raw icon instance dropped into a layout skips the
  wrapper and can't be recoloured or swapped consistently.
- **Fixed or placeholder: decide first, because it changes what you build.**
  - **Placeholder**: the icon INSTANCE gets an `INSTANCE_SWAP` prop with the icon library as
    preferred values, plus a `Show icon` boolean on the outer frame.
  - **Fixed**: **no swap prop.** The icon is part of what the component means.
- **Where fixed, choose by description, never by name.** `trash`, `x` and `circle-x` are three
  different jobs. Search by meaning with `search_design_system`, shortlist, read the descriptions,
  then pick. **Filter results to the component library**: unrestricted search returns icons from
  legacy libraries. **If nothing fits, say so** rather than forcing the nearest match.
- When a consumed component has its own icon, turn it OFF and document that the host's icon is
  the only icon affordance.

### Slots

- **Slots are scriptable**: `component.createSlot()` (verified 2026-08-27).
  1. Build the variant with a temporary holder FRAME containing the default content.
  2. `const slot = comp.createSlot(comp.children[0])`. This APPENDS an empty SLOT after that
     node; it doesn't convert it. It always lands at the component's top level: to nest it,
     move it with `insertChild`, and it stays a slot.
  3. `slot.name = 'items'` (the layer name becomes the prop name), then
     `comp.editComponentProperty(slotKey, { name, preferredValues, slotSettings })`.
  4. The new SLOT has `layoutMode:'NONE'`. Set auto-layout, sizing and gap yourself, move the
     holder's children in, and `holder.remove()`.
  5. Same-named slots across variants do **not** reliably merge into one prop (2026-10-02). Check
     the set's slot props, re-point each variant's slot to one property with
     `componentPropertyReferences`, then delete the extras.
- **Cloning a variant degrades its slot to a frame** and drops its property links. Re-promote it
  (`design_foundations` § Working in Figma via MCP, Cloning and swapping).
- **Filling a slot in an instance is structural**: `setProperties` can't set a SLOT prop, but the
  slot node accepts `appendChild` (`design_foundations` § Slots).
- **Removing inherited slot placeholders from an instance works** (2026-10). Before 2026-09 it
  didn't: they couldn't be removed, hidden or reset, and `displayEmptyByDefault: true` didn't
  suppress them. Try a removal on one throwaway instance first. If it fails, try hiding them
  (`visible=false`, matching by prop value), which has worked in some sessions. Failing both,
  use build order: empty the slot in every variant, create and fill all instances, then re-add
  the placeholders. Scripting only; the UI is unaffected.
- Rows dropped into a slot arrive `FILL/FIXED`. Set `layoutSizingVertical='HUG'`.
- **Preferred values that cover only part of what the CRD allows** (the rest aren't in the
  library yet): record which are missing in the CRD's Notes via `ds-crd-update`. When you design
  one of those missing components, search dependent CRDs' Notes for it and add it to their slots.
- Never write "a button". Name the component and its permitted variants.

## 5. Composition

- **Follow `design_foundations` § Building layouts**: auto-layout everywhere, no spacer frames,
  clip only where genuinely needed, strip frame defaults, gap and padding bound to tokens. In a
  component, an unstripped white fill silently clones into every variant, and a spacer leaks
  into every instance. Moving a visibility binding (§ 4) often removes the need for one.
- **Never wrap `createInstance()` in a font-retry loop.** Each failed attempt leaves a stray
  instance behind. Load the fonts first, then create once.
- Positioning is owned by the parent. A child never specs its own placement.
- Consume existing components rather than redrawing; record any forced settings in the CRD.
- Behaviours Figma can't express: build the nearest approximation, and spec the real rule via Dev
  Mode annotations (`node.annotations = [{ labelMarkdown }]`) and the component description.
- **A single nested override can't be reset by script** (`design_foundations` § Working in Figma
  via MCP). Set what renders correctly and ask the person to right-click → **Reset changes**.

## 6. Decisions go back to the CRD as they happen

Design decides most of a CRD's content, verbally. Captured only at handoff, it arrives filtered
through memory. Batch decisions, but don't defer them to the end. Log them via `ds-crd-update`,
one line each, per `crd_how_to` § Keeping it tight. The build story belongs on the context card,
not here.

- **An `[open]` marker resolved**: promoted, moved to Documentation guidance, or cut. All three are
  logged in Design decisions with the why.
- **A decision someone could plausibly propose the opposite of.** Interview churn isn't one.
- **A rejected option** → Ruled out, one line and its reason.
- **A contract statement that turns out false**: correct it and say so. A "small" change that
  invalidates several statements probably wanted to be a bigger amendment.
- **An open question answered**: remove the row and record the answer.

Adjustments *forced* by the design (padding, optical corrections) are reported in chat, never
written into the CRD. They live in Figma and the tokens.

## 7. Verification (non-negotiable)

- **Resolve every binding across every mode axis** (`design_foundations` § Mode axes) with
  `resolveForConsumer`, not just "a variable is bound". A binding to an old local or single-mode
  collection passes the existence check and resolves wrong everywhere but one mode.
- **Verify paints by rendering.** A binding can be present and still paint nothing; the tell is
  `paint.color` reading white (`design_foundations` § Working in Figma via MCP). Fix by
  recreating the node in the importing script.
- **Whole-page token sweep: every property that can take a variable has one.** Check every node
  on the page, including variants, doc frames and the inside of consumed instances, against
  everything Figma lets you bind:
  - **Paints**: every fill and stroke. Gradient stops count: each stop is bound to a colour
    variable.
  - **Spacing**: padding on all four sides, and gap (both gaps on a grid or wrapping row).
  - **Shape**: corner radius (per corner when the corners differ) and stroke weight (per side
    when the sides differ).
  - **Sizes**: width and height where they're fixed, plus min and max width and height.
  - **Effects**: shadow and blur values, or an effect style where `design_foundations` § Effects
    and icons applies effects that way.
  - **Opacity**, and anything else the node's `boundVariables` can hold.

  **Anything left raw is a finding to fix**, except zero values and the CRD's exemptions. An
  exemption is a value code calculates, or one whose value or unit Figma doesn't support yet. Each
  is recorded in the CRD via `ds-crd-update` and flagged with a Dev Mode annotation on the node.
- **Whole-page style sweep**: every text node on the page has a text style and a token fill,
  including labels you cloned or moved, not just nodes you created. Text content is in config
  § Writing → `text_case`, with casing applied by the style. No glyph icons (§ 4).
- **Clip sweep**: every frame on the page, doc frames and variants included, has clip content
  off, unless the clip is a deliberate effect (`design_foundations` § Building layouts) **and**
  annotated with why. An unannotated clip is a finding even when it looks right: it hides
  overflow rather than fixing it, and the next person can't tell it was meant.
- **No forbidden bindings**: no direct lower-tier bindings beyond those `design_foundations`
  § Component tokens allows, no raw hex or detached fills, no variables or text styles from a
  previous system, no deprecated tokens (config § Tokens → `deprecated_name`; rebind to the
  replacement). **Sweep by collection, not just by name**: list the variable collection of every
  binding on the page; any deprecated or retired collection is a finding even when the token
  name looks current.
- **After creating or restructuring variants, rerun the token sweep over every variant you
  touched**, descending into consumed instances you swapped or restructured (their resolved
  values, not just `mainComponent`). Targeted binding checks miss nodes you didn't target, and
  skipping instance interiors is how stale override bindings hide.
- Temp swatch frames with explicit variable modes, `screenshot()`, then **delete them**.
- **Contrast results never go in a CRD.** Check contrast across the modes that must pass
  (config § Accessibility → `contrast_must_pass`) and raise anything that fails in chat.
- **After a rejected or interrupted write, re-read before re-running** (`design_foundations`
  § Working in Figma via MCP).
- Return created and mutated node IDs from every script.

## 8. Handoff

- Context card current: Status matches `status:`, and a changelog entry added.
- **Dev Mode annotations** for the values and rules code needs. **Annotate a repeated structure
  once**, on the first or default variant, never on every variant; explain it in context on the
  page (a short note beside the set) instead.
- **Motion**: if the component moves, the animations in the motion frame carry the build values,
  one per movement the CRD lists (`documentation-frames.md` § Motion frame). One Dev Mode
  annotation on the set says what moves, the motion tokens used and the reduced-motion behaviour,
  and links each animation frame. No durations or easings in it: the animation is the source. The
  CRD describes the motion in words only.
- Update the CRD via `ds-crd-update`. Props tables must match the built component.
- Remind config § Roles → `publisher` to publish the library when consumers need the changes.
- **Last, ask whether design is done.** If yes, `ds-crd-update` sets `design_specs: complete`.
  **For an amendment, closing design includes the fold**: hand to `ds-crd-update` in the same
  pass (pre-fold sweep, rewrite section 1, delete superseded lines and the amendment block, write
  Migration). Then hand to `ds-crd-write` for a short section 4 interview (patterns, gotchas,
  content guidance) while the design is fresh. Nothing else prompts any of these.
- **Then `ds-ready-for-dev`** checks the whole hand-off and moves the tickets.

## When to use

- Setting up a new component page, or building a component in the library
- Adding a variant or size axis
- Wiring component tokens end to end, raw values through to the library binding
- Slots, props, or context-card documentation
