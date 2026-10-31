---
name: ds-asset-normalize
description: Batch-normalize app/logo SVG assets in the Figma component library into the standard icon format — square frame, ratio locked, height bound to the app-icon sizing token, artwork set to SCALE both directions — then convert each to an individual component named by the library's app-icon convention and arrange them in a grid on an assets-page artboard. Use when cleaning up pasted-in SVG logos on the assets page, normalizing icon frames to consistent sizing, or componentizing a batch of app logos.
---

# Asset normalize — batch SVG/logo cleanup → components

Optional. Normalizes a set of logo/asset nodes (frames, groups, in any messy state) into one
consistent format, then componentizes and renames them.

**File**: config § Figma → `component_library`. Assets typically live on the page named in config
§ Figma → `assets_page`.

## Target end state (per asset)

Read the target from **two known-good existing app icons** before starting, and confirm it matches
`design_foundations` § Effects and icons. The usual shape:

- `COMPONENT` node (converted from frame), named per config § Figma → `app_icon_name` (e.g.
  `icons/app-{name}`, lowercase kebab)
- Square, at the app-icon size (`SIZE` below) named in § Effects and icons
- `targetAspectRatio` locked (via `lockAspectRatio()`)
- `boundVariables.height` → the app-icon sizing token from § Effects and icons. **Bind only what
  the known-good siblings bind** — commonly height only, not width.
- Every descendant with `constraints` set to `{ horizontal: 'SCALE', vertical: 'SCALE' }`
- `clipsContent: true`

## Prerequisites

1. Load the `figma:figma-use` skill first (mandatory before `use_figma`).
2. The sizing token is usually a **published remote variable** — bind via
   `figma.variables.importVariableByKeyAsync(key)`, never by ID lookup in local collections. Get
   the key with `search_design_system`, or read `boundVariables.height` off a known-good sibling.
   If the import fails, the token probably isn't published (`design_foundations` § Publish
   gates).

## Workflow

### Step 1 — Inventory (read-only)

Get the target node set from the person (a selection, a region, or "everything on the page"). Run
a read-only scan classifying each candidate:

```js
// currentPage must be the assets page: setCurrentPageAsync first if needed
const nodes = figma.currentPage.selection.length
  ? [...figma.currentPage.selection]
  : figma.currentPage.children.filter(n => /* region/filter from the person */ true);

const safe = (n,p) => { try { return n[p]; } catch(e){ return undefined; } };
return nodes.map(n => ({
  id: n.id, name: n.name, type: n.type,
  w: Math.round(n.width*100)/100, h: Math.round(n.height*100)/100,
  square: Math.abs(n.width - n.height) < 0.5,
  ratioLocked: !!safe(n,'targetAspectRatio'),
  heightBound: !!(safe(n,'boundVariables')||{}).height,
  childNames: (n.children||[]).map(c => c.name),
}));
```

Classify:

| State | Action |
|---|---|
| FRAME (any size/shape) | normalize (Step 2) |
| GROUP | wrap in frame first (Step 2a), then normalize |
| COMPONENT already | normalize in place, skip componentize, just rename |
| INSTANCE | **flag to the person, do not process** — it's a copy of a component that lives elsewhere; detaching or duplicating is their call |
| Anything nested inside another candidate | process only the outermost |

Report the inventory and planned names (Step 4) **before mutating** if the batch is large or any
names are uncertain.

### Step 2 — Normalize each asset

The tested per-asset recipe. **Order matters** (resize resets things; constraints must be SCALE
*before* the final resize):

```js
const SIZE = /* app-icon size from design_foundations § Effects and icons, e.g. 96 */;
const SIZING_KEY = /* the sizing token's variable key */;
const frame = await figma.getNodeByIdAsync(ID);

// 1. All descendants scale in both directions (recurse — nested frames/vectors alike)
const setScale = (n) => {
  if ('constraints' in n) { try { n.constraints = { horizontal: 'SCALE', vertical: 'SCALE' }; } catch(e){} }
  if (n.children) n.children.forEach(setScale);
};
frame.children.forEach(setScale);

// 2. Square it WITHOUT moving/scaling children, then center the artwork
const w = frame.width, h = frame.height, square = Math.max(w, h);
frame.resizeWithoutConstraints(square, square);
const dx = (square - w) / 2, dy = (square - h) / 2;
for (const c of frame.children) { c.x += dx; c.y += dy; }

// 3. Scale to SIZE — SCALE constraints scale artwork uniformly (frame is square, so no distortion)
frame.resize(SIZE, SIZE);

// 4. Lock ratio, clip, bind height token
frame.lockAspectRatio();
frame.clipsContent = true;
const sizingVar = await figma.variables.importVariableByKeyAsync(SIZING_KEY);
frame.setBoundVariable('height', sizingVar);
```

```js
// 5. CRITICAL — resize() with SCALE constraints scales GEOMETRY ONLY.
// strokeWeight, effect radii (blurs/shadows), and RECTANGLE cornerRadius do NOT scale.
// Multiply them manually by the same factor or scaled icons render with wrong-weight
// strokes and oversized blur halos (this bit a real batch: 192→96 icons kept
// 12px strokes and 12–57px LAYER_BLURs that should have halved).
const f = SIZE / square; // the scale factor applied by resize()
if (Math.abs(f - 1) > 0.001) {
  const fixNonGeometry = (n) => {
    try { if (n.strokes && n.strokes.some(s => s.visible !== false) && typeof n.strokeWeight === 'number' && n.strokeWeight > 0) n.strokeWeight *= f; } catch(e) {}
    try { if (n.type === 'RECTANGLE' && typeof n.cornerRadius === 'number' && n.cornerRadius > 0) n.cornerRadius *= f; } catch(e) {}
    try {
      if (n.effects && n.effects.length) {
        n.effects = n.effects.map(e => {
          const c = JSON.parse(JSON.stringify(e));
          if (typeof c.radius === 'number') c.radius *= f;
          if (typeof c.spread === 'number') c.spread *= f;
          if (c.offset) c.offset = { x: c.offset.x * f, y: c.offset.y * f };
          return c;
        });
      }
    } catch(e) {}
    if (n.children) n.children.forEach(fixNonGeometry);
  };
  frame.children.forEach(fixNonGeometry);
}
```

Notes:

- **Verify against source SVGs when available** (render them, e.g. with
  `qlmanage -t -s 192 -o <dir> <file>.svg` on macOS, and compare screenshots). Bounding-box checks
  alone miss stroke/blur/radius scaling defects, and mask-group content legitimately overflows the
  frame, so raw bbox overflow is not proof of breakage.
- The recipe is idempotent: on an already-square frame, dx and dy are 0.
- If artwork visibly overflows or sits off-center afterward, the vectors' original offsets were
  asymmetric — recenter by bounding box instead: compute min/max of child bounds, shift so the
  artwork block is centered.

#### Step 2a — GROUP inputs

Wrap first, then run Step 2 on the wrapper:

```js
const group = await figma.getNodeByIdAsync(ID);
const parent = group.parent, gx = group.x, gy = group.y;
const frame = figma.createFrame();
frame.name = group.name;
frame.x = gx; frame.y = gy;
frame.resizeWithoutConstraints(group.width, group.height);
frame.fills = [];
parent.appendChild(frame);
frame.appendChild(group);
group.x = 0; group.y = 0;
figma.ungroup(group); // children become direct children of frame
```

### Step 3 — Componentize

After normalize (so the component inherits the final geometry):

```js
const component = figma.createComponentFromNode(frame); // in place, same position
```

Then **verify the conversion preserved** `targetAspectRatio`, `boundVariables.height`, and child
constraints — re-apply Step 2's item 4 if any were dropped.

### Step 4 — Rename per `app_icon_name`

Deduce `{name}` per asset, in priority order:

1. **The asset's own layer name**, cleaned (rules below) — e.g. `m365_48x1 1` → `m365`,
   `Skype_logo_(2019–present) (1) 1` → `skype`.
2. **Descendant layer names** that aren't generic (`Vector`, `Group`, `path`, `Layer_1`,
   `Component N`, `Frame N`).
3. **Screenshot + visual identification** — `await node.screenshot()` and recognize the logo.
4. Still unknown → collect them and, at the end, list them with node links, then **ask one at a
   time** (`.claude/CLAUDE.kit.md` § Asking questions); never guess-name.

Cleaning rules:

- lowercase; spaces/underscores → hyphens; strip diacritics/symbols
- strip: size suffixes (`_48x1`, `48px`, `@2x`), file junk (`.svg`, `(1)`), the words
  `logo`/`icon`, parenthetical qualifiers (`(2019–present)`), Figma paste-dedup trailing digits
  (` 1`, ` 2`)
- use the **product name alone**, no vendor prefix, matching existing siblings (`teams`, not
  `microsoft-teams`)
- check for collisions with existing app-icon components before assigning; on collision, flag it

### Step 5 — Assemble onto an artboard

Once all components are normalized and renamed, collect them into a tidy grid on an artboard
matching the other artboards on the assets page.

**Match the existing artboard convention — read it, don't assume it.** Inspect one or two
existing asset artboards and copy their: position relative to the rightmost artboard and gap,
auto-layout direction, padding, `itemSpacing`, `cornerRadius`, hug sizing, fill (and the variable
it binds — import by key, then `figma.variables.setBoundVariableForPaint(paint, 'color', v)` and
reassign fills), and the section-header component instanced as the first child, if any (config
§ Figma → `section_header_component`; `importComponentByKeyAsync` → `createInstance()`). Override
the header's text layers using the canonical font-load recipe (load each text node's current fonts
via `getStyledTextSegments(['fontName'])` before setting `characters`), with text in config
§ Writing → `text_case`.

**Before creating a new artboard, check for an existing near-empty one** that may be the intended
destination. Ask the person which to use if ambiguous.

**Grid**: inside the artboard, one wrapping auto-layout frame holding the components:

```js
const grid = figma.createAutoLayout('HORIZONTAL', { name: 'app-icons', itemSpacing: 48 });
grid.layoutWrap = 'WRAP';
grid.counterAxisSpacing = 48;
grid.fills = [];
artboard.appendChild(grid);
grid.layoutSizingHorizontal = 'FIXED';
grid.resize(HEADER_WIDTH, grid.height); // match the header's width, e.g. 960 → 7 per row at 96px
grid.layoutSizingVertical = 'HUG';
// Sort components alphabetically by name, then reparent:
for (const comp of sortedComponents) grid.appendChild(comp); // appendChild moves them
```

Notes:

- Bind the grid's gaps to spacing tokens if the siblings do (`design_foundations` § Spacing).
- Components can live inside frames; `appendChild` moves them without breaking existing instances.
- Verify after reparenting that each component is still `SIZE`×`SIZE` with
  `boundVariables.height` intact (auto-layout must not stretch them — children default to FIXED
  sizing, keep it).
- Screenshot the finished artboard for a final visual check.

### Step 6 — Batching & verification

- **Max ~4–5 assets per `use_figma` call** (each asset ≈ 3–4 logical ops; obey the ≤10-ops rule).
  Loop batches; return mutated/created IDs from every call.
- After each batch: spot-check one `screenshot()`; every ~3 batches or at the end, run the Step 1
  scan again over processed IDs and assert: type COMPONENT, `SIZE`×`SIZE`, ratioLocked,
  heightBound, all descendants SCALE, name matches the `app_icon_name` pattern.
- Final report: table of `old name → new name`, plus flagged items (instances, unresolvable names,
  collisions), plus the artboard node ID.

## Gotchas

- `resize()` before `lockAspectRatio()` — resize updates/ignores the ratio; lock last.
- `resize()` scales geometry only — always run the stroke/effect/cornerRadius factor fix (Step 2
  item 5) after any non-1× resize.
- `resizeWithoutConstraints` is the key to squaring without distorting: it leaves children
  untouched.
- INSTANCE nodes: never `createComponentFromNode` on an instance-derived tree without the person's
  sign-off.
- Remote variable binding **requires** `importVariableByKeyAsync` — `getVariableByIdAsync` with a
  stale local ID fails silently across files.
- Creating components changes the library: remind config § Roles → `publisher` to **publish**
  before consumers see them.
