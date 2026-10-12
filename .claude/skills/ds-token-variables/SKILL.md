---
name: ds-token-variables
description: Orientation and working rules for Figma variable work in the design system's token files — the tier pipeline, collections and mode axes, density presets, token families and deprecation, naming and changelog rules, publish gates, contrast verification and census. Use when creating, renaming, deprecating, editing or auditing variables in the token FILES themselves, or when working out how a token resolves per brand, theme and breakpoint. Design system team only. For component dimension tokens wired into a library component, ds-design-component § 2 covers the flow.
---

# Token variables (Figma token files)

Working in the token files themselves. **Design system team only**: contributors building
components never need this skill. Requires `figma-use` for all reads and writes.

The system's own architecture (files, tiers, collections, modes, naming) lives in
config § Paths → `design_foundations`. **Read it first.** This skill holds the process and the
rules that apply whatever the architecture. It carries no variable counts; when a count
matters, take it live with the § 5 census.

## 1. Files, collections, modes

From `design_foundations` § Files, § Token tiers and § Mode axes:

- **Which file holds which tier**, and the alias direction between them. Tokens alias downward
  through the tiers; components bind only the tier `design_foundations` § Token tiers names.
- **Each collection's modes** (§ 2).
- **The library consumes published values only.** Downstream files see a change only after it
  is published (§ 4).
- **Export markers.** If config § Tokens → `no_export_marker` is set, a mode or variable carrying
  it is excluded from the code token export.

**Mode-driven responsive values**: if your system resolves breakpoints by mode, the raw
per-breakpoint values usually live flat in a lower tier, and one semantic variable per property
collapses them into a single variable with one mode per breakpoint. Components bind that one
variable; switching breakpoint is setting the frame's mode. Never bind several variables and
toggle visibility.

## 2. Mode axes

A token can vary along several independent mode axes (`design_foundations` § Mode axes): brand,
theme and breakpoint, for example. A frame or component picks each one separately, and the same
semantic structure resolves differently per mode. **Check a change in every mode of every axis
it touches**, not just the default.

**Never assume light or dark from a mode's name.** A brand's base can be dark. Read the resolved
values.

### Density *(if your system has density presets)*

Density usually rides the brand axis: a lower tier holds every density family in each tier
(e.g. spacious, standard, compact), and each brand mode aliases one tier. `design_foundations`
§ Spacing names your families and which brand takes which tier.

- **Each brand takes its full preset across every family.** Don't mix tiers within a brand. A
  new brand mode picks one tier and aliases it everywhere.
- **Tier tally (check after any density or brand-mode edit).** For each brand mode, walk every
  variable that aliases a density variable, and count the tier in the target's name, grouped
  by family. It passes when each mode lands entirely on its tier and every mode has the same
  count per family. A mixed family or uneven count is a defect.

## 3. Token families

The source of truth is the **published** semantic library, queried with
`search_design_system`. Don't read the exported JSON to decide what exists: it lags behind
publishing and may be keyed on code syntax rather than the variable name
(`design_foundations` § Finding tokens).

- **Families travel together.** A colour that only works with its partners (a status fill, its
  on-fill ink and its border) is created, changed and deprecated as a set. Never change one
  without checking the others still pair.
- **System rules hold.** Tokens that `design_foundations` § System rules constrains (must not vary
  by brand, for instance) keep to that constraint in every mode.
- **Gaps are real: flag them, never fake them.** A token missing in one mode or family is a
  finding, not something to fill with a neighbour's value.
- **Deprecated**: config § Tokens → `deprecated_name`. A leaf rename with values frozen; nothing
  new binds to it.

## 4. Working rules for token edits

- **Publish gate.** Downstream files see only published values. After edits, pause and ask
  config § Roles → `publisher` to publish before any import, binding or demo can reflect them.
  `importVariableByKeyAsync` fails outright on an unpublished variable, so "not found" usually
  means "needs publishing" (`design_foundations` § Publish gates).
- **Contrast before finishing** any value change. Check every combination in config
  § Accessibility → `contrast_must_pass` against config § Accessibility → `standard`. Brands in
  `contrast_latitude` should be made passable, not blocked on. Present borderline cells to the
  person; don't decide silently.
- **Names follow values.** The value is the source of truth: if a number in a name disagrees
  with it, rename to match, and update the variable's code syntax in the same edit. A rename
  never updates code syntax, and exporters often read code syntax rather than the name.
- **Mismatch scan (after any value or name change).** For each variable whose name ends in a
  number, compare that number with the resolved value. Allow for your naming encodings
  (`design_foundations` § Naming): negatives, decimals written with a leading zero, ratios ×100,
  and float noise (0.8999…). Then check each code syntax still matches the current name.
- **Changelog entry on every edit.** If your token files carry a context card (config § Figma →
  `context_card`, same format as a component page's, `ds-design-component` § 1), add one dated
  entry per edit to the file you changed: what and when, not why. If they don't, put the entry
  in the PR or commit that records the change.
- **Export impact → work item.** If the edit changes an exported name or value, raise a work
  item for its export impact through `ds-ticket`, naming what changed. Changes inside
  non-exported modes skip the export, but anything built downstream from them may still need
  regenerating; say so when it applies.
- **Deliberate variance is not a defect.** Some complex component tokens alias a lower tier
  directly by design. Record each one in `design_foundations` § Component tokens, so later
  audits don't report it.
- **Set `variable.scopes` explicitly on creation.** Match sibling scopes and code syntax
  patterns.
- **Imported-but-unused variables get pruned between scripts**, and a dead reference silently
  unbinds paints. Null-check and throw; import and bind in the same script
  (`design_foundations` § Working in Figma via MCP).
- **New component dimension tokens** follow `ds-design-component` § 2.

## 5. Census (when counts or structure matter)

```js
const colls = await figma.variables.getLocalVariableCollectionsAsync();
const vars = await figma.variables.getLocalVariablesAsync();
return colls.map(c => ({
  name: c.name, modes: c.modes.map(m => m.name), total: c.variableIds.length,
  groups: vars.filter(v => v.variableCollectionId === c.id)
    .reduce((g, v) => { const k = v.name.split('/').slice(0, 2).join('/'); g[k] = (g[k] || 0) + 1; return g; }, {})
}));
```

To trace a value, walk `valuesByMode` → `VARIABLE_ALIAS` → `getVariableByIdAsync` per mode.
Alias chains cross into lower-tier files; remote variables resolve fine read-only.

## When to use

- Creating, renaming or deprecating variables in any token file
- Changing token values or mode columns (brand, theme, breakpoint)
- Adding a mode, or extending a token family
- Working out how a token resolves per brand × theme × breakpoint
