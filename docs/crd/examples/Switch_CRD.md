---
id: switch
name: Switch
version: 0.1.0
format: 2
lifecycle: requested
tier: core
composition: basic
category: forms
status:
  requirements: in_progress
  design_specs: not_started
  implementation: not_started
  docs_guidance: in_progress
figma:
docs_site:
related_crds:
  dependency: []
  precedent: [button, text-field, selection-tile]
last_updated: 2026-10-06
---

# CRD: Switch

<!-- Worked example for the invented Acme design system: a CRD early in its life, requirements
drafted and design not started. Every reference is illustrative. -->

---

# 1 · Requirements

## Purpose

A binary control for a single setting the user flips on or off. It solves the case where the
current state matters more than the act of changing it — a setting read far more often than it
is edited.

The Switch is the control only. The row it usually sits in — label at the reading-start edge,
control at the reading-end edge — is a separate component still to be specified.

## Scope

**In scope:**

* A single on/off value
* One size
* Resting, hover, focus and disabled states across both values
* Immediate-effect and form-staged use, with no difference between them
* RTL mirroring

**Out of scope:**

* The label, description and row layout around the control — a Switch Row component will cover
  this; it does not exist yet
* Staged multi-select choices in a form — see Chip Tag for the interactive filter case
* Triggering an action rather than setting a value — see Button
* Mutually exclusive choices from a set — see Selection Tile for the visual pick-one
  case (requested, not built). A radio group and a segmented control are still uncovered
* A third or indeterminate value — see Ruled out

## Anatomy

* **track** — the pill-shaped container. Its fill carries the value
* **handle** — the circular indicator inside the track. Its position carries the value

Both parts are always present. The Switch has no optional parts.

## Functional requirements

* F1: The Switch represents a single binary value, on or off
* F2: The Switch has no indeterminate or third value
* F3: Activating the Switch inverts its value
* F4: The handle sits at the reading-start edge of the track when the value is off, and at the
  reading-end edge when the value is on
* F5: The track fill differs between the on value and the off value
* F6: The Switch ships in exactly one size
* F7: The Switch renders no text and no icon
* F8: The Switch's appearance and behaviour are identical whether the consumer applies the value
  immediately or submits it with a parent form
* F9: The Switch exposes no pending or in-flight treatment

## States

* S1: off — the resting appearance when the value is off
* S2: on — the resting appearance when the value is on
* S3: hover — pointer over the track
* S4: focus — keyboard focus, with a visible focus indicator
* S5: disabled — non-interactive
* S6: hover and focus each apply to both the on and the off value
* S7: disabled applies to both the on and the off value, and continues to show which value is set

## Behaviour

* B1: Clicking or tapping anywhere on the track inverts the value
* B2: Space inverts the value while the Switch has focus
* B3: Enter does not invert the value
* B4: The Switch does not change value on hover or focus alone
* B5: disabled prevents the value changing by pointer and by keyboard
* B6: The handle animates between its off and on positions rather than jumping
* B7: Under reduced motion the handle moves to its new position without animating
* B8: Under RTL the Switch mirrors, so the reading-start and reading-end edges swap and the
  handle travels in the opposite direction

## Content

## Accessibility

* A1: The Switch exposes the `switch` role
* A2: The Switch exposes `aria-checked="true"` when on and `aria-checked="false"` when off
* A3: The Switch never exposes `aria-checked="mixed"` — the `switch` role has no mixed value
* A4: The Switch takes its accessible name from the consumer
* A5: A value change is announced by the change of `aria-checked`, with no separate live-region
  announcement
* A6: The track and handle are decorative to assistive technology; the value reaches assistive
  technology through `aria-checked` only
* A7: The handle position conveys the value independently of the track fill, so the value is not
  signalled by colour alone

## Integration

**Composition rules:**

* I1: The Switch accepts no children and no slotted content
* I2: The Switch does not render a label; the consumer supplies the accessible name
* I3: The Switch's width and height are fixed by its single size and do not fill the container

**Children:**

None.

**Used in:**

* Switch Row — planned, not yet specified
* Settings panels, modals, and form layouts

## Ruled out

* **Icon in the handle** (Material 3, Atlassian) — the handle's position already conveys the
  value without relying on colour, so an icon adds brand and build cost for no accessibility gain
* **On/off text inside the track** (Carbon) — localisation and width cost, and redundant against
  the position cue
* **`active` state** (Material 3, Spectrum) — no distinct design treatment, on the Text Field
  precedent that the system drops states it does not draw
* **Pending / in-flight state** — whether a change is unconfirmed is the consuming design's
  problem, not the Switch's
* **A second size** — no evidenced need
* **A third or indeterminate value** — the `switch` role has no mixed value, and a tri-state
  need is a different control
* **Enter as an activation key** — Space only, matching the `switch` role in the WAI-ARIA
  Authoring Practices and native checkbox semantics

## Open questions

| Question | Owner | By | Status |
|---|---|---|---|
| Is the on-state track the brand accent, or a fixed semantic colour? A semantic colour would fix the on state across brands and change which tokens it consumes. | Design lead | Before the Switch Row is designed | open |
| Which semantics — `<input type="checkbox" role="switch">` or `<button role="switch">`? Form participation and whether disabled stays focusable both follow from this. | Development | Before dev starts | open |

## Notes

* The setting-row pattern (label at the reading-start edge, control at the reading-end edge) is
  undesigned. The Text Field CRD records it as the counterpart to the label-top form field;
  the Switch is the first control that pattern needs.
* No Figma component set exists yet.
* The Switch has one fixed size and no text, so unlike Button it cannot grow into the
  global minimum touch target on its own — the hit area will always need to exceed the visible
  track. A design constraint on the single size, not a Switch-specific accessibility rule.
* Contrast is validated per theme and per brand, not per variant. The on-state track's contrast
  against surrounding surfaces cannot be promised until the on-state colour question closes.

---

# 2 · Design specifications

## Properties in Figma

| Property | Type | Default | Options | Description |
|---|---|---|---|---|

## Design decisions

* 2026-09-03: Named **Switch**, not Toggle — WAI-ARIA calls the role `switch`, so the component
  name and the role it exposes match.
* 2026-09-03: **Control only.** The label row is deferred to a Switch Row component rather than
  built into the Switch, so the control stays reusable outside a settings row.

---

# 3 · Implementation

## Implementation decisions

## Dev notes & deviations

---

# 4 · Documentation guidance

## Usage

**When to use:**

* For a single independent setting, where the current state matters more than the act of
  changing it

**When not to use:**

* Staged multi-select filters — use Chip Tag's selected state
* Triggering an action — use Button
* Picking one option from a set — use Selection Tile

**Patterns:**

**Content:**

* Label voice, casing and length belong to the consumer placing the Switch, and to Switch Row
  once it exists

## Gotchas

