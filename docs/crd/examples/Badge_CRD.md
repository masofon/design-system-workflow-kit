---
id: badge
name: Badge
version: 2.0.1
format: 2
lifecycle: built
tier: core
composition: composite
category: feedback
status:
  requirements: complete
  design_specs: complete
  implementation: in_progress
  docs_guidance: complete
figma: https://www.figma.com/design/EXAMPLEKEY/Acme-Components?node-id=0-1
docs_site: https://storybook.acme.example/?path=/docs/feedback-badge--docs
related_crds:
  dependency: [status-indicator]
  precedent: []
last_updated: 2026-10-06
---

# CRD: Badge

<!-- Worked example for the invented Acme design system; every token, link and decision is
illustrative. -->

---

# 1 · Requirements

## Purpose

A small, non-interactive label — usually a count, a status or a short piece of text — that draws the eye to something needing attention, such as unread messages or new alerts, without crowding the layout.

## Scope

**In scope:**

* Text, icon and status indicator content, including numbers and abbreviated counts
* Sentiment and accent colours, each in outline and filled treatments
* Two sizes

**Out of scope:**

* Interactive labels for filtering or navigation — use Chip Tag
* A status dot on its own, without a label — use Status Indicator
* Shadows or other context styling — applied by the consuming component

## Anatomy

* **Container** — the badge's box, carrying its fill and border
* **Icon** (optional) — a leading icon
* **Status indicator** (optional) — a Status Indicator in its own slot, in place of the icon
* **Label** (optional) — the text
* **Custom content** (optional) — consumer-supplied content for layouts the label and icon cannot express; renders after them

## Functional requirements

* F1: The badge is non-interactive: it has no hover, pressed, click or focus behaviour of its own.
* F2: The badge renders at least one of label, icon or custom content; with none of them it renders nothing.
* F3: Content is one of: label only, icon only, icon and label, or status indicator and label.
* F4: The icon is leading only.
* F5: The status indicator and the icon never appear together.
* F6: The label never wraps; it truncates with an ellipsis when it exceeds the available width.
* F7: When the icon fails to load, the badge shows the label alone, or nothing if there is no label, with no loading state.
* F8: Every colour — success, warning, error, info, neutral, neutral-mid, neutral-dark, accent 1, accent 2, accent 3 — is available in both outline and filled treatments.
* F9: The badge comes in two sizes, l and sm.
* F10: The status indicator keeps its own sentiment colour; the badge ink is never applied to it.
* F11: Status indicator and label is available on every colour, in both treatments and both sizes.
* F12: The status indicator's sentiment — positive, negative, warning, info or neutral — is chosen independently of the badge colour.
* F13: The status indicator's size follows the badge size.

## States

* S1: The badge has one state, default; it has no interactive or disabled states.

## Behaviour

* B1: The icon stays at its size's fixed dimension across breakpoints.
* B2: The badge has no animation or transitions.

## Content

* C1: The label shows at most 20 characters; a longer label truncates with an ellipsis.

## Accessibility

* A1: The icon is decorative and hidden from the accessibility tree; the label, or the accessible-name override, names the badge.
* A2: An icon-only badge requires an accessible-name override.
* A3: The accessible-name override is honoured on every content variant; when supplied the badge takes `role="img"` and the override replaces the visible text in the announcement.
* A4: Without an override the badge carries no role.
* A5: The status indicator is hidden from the accessibility tree (`aria-hidden="true"`); the label carries the status.
* A6: The nested status indicator keeps no accessible name of its own.
* A7: The badge sets no `role="status"` and is not a live region.

## Integration

**Composition rules:**

* I1: A badge never contains another badge.
* I2: When a consumer wraps a badge in an interactive element, the wrapper owns all interaction, states and focus (F1).

**Children:**

* Icon
* Status Indicator

**Used in:**

* Notifications
* Profile pages and user lists
* Comment threads and chat
* Cards and panels
* Tables

## Ruled out

* A neutral-only status indicator — the dot is a status in its own right, so it pairs with any badge colour (F11).
* A border on filled neutral instead of a new fill — it would only duplicate outline neutral.
* Dropping filled neutral — breaking for consumers already using it.
* Border tokens that duplicate the inks — two names for one colour, free to drift apart.

## Open questions

| Question | Owner | By | Status |
|---|---|---|---|

## Notes

---

# 2 · Design specifications

## Properties in Figma

| Property | Type | Default | Options | Description |
|---|---|---|---|---|
| label | Text | label | | |
| icon | Instance swap | | Icon library | |
| fill | Variant | outline | outline, fill | |
| color | Variant | neutral | success, warning, error, info, neutral, neutral-mid, neutral-dark, accent 1, accent 2, accent 3 | |
| size | Variant | l | l, sm | |
| content | Variant | icon-and-text | icon-only, text-only, icon-and-text, indicator-and-text | indicator-and-text exposes the nested Status Indicator: its Variant sets the sentiment (F12); leave its Size, which follows `size` (F13) |

## Design decisions

* 2026-08-25: Four neutral `l` icon variants carry the ink token on the icon vector rather than a wrapper layer — inserting a wrapper would have reset icon-swap overrides in consuming files.
* 2026-08-28: The status indicator has its own slot rather than using the icon slot — the icon slot recolours its contents to the badge ink, which makes the indicator vanish.
* 2026-10-06: The status indicator's sentiment is set on the exposed nested Status Indicator, not a swap property (F12) — a swap property holds only one default sentiment per size.
* 2026-10-06: Filled neutral, neutral-mid and neutral-dark each take a distinct bold fill — a subtle neutral fill matches the page background in every brand.
* 2026-10-06: An outline badge's border takes its ink colour, so it matches the text.
* 2026-10-06: The `sm` status indicator uses the thin border token, defined for that size.
* 2026-10-06: A5–A7 — the label already names the status, so the dot stays decorative, like the icon.

---

# 3 · Implementation

## Implementation decisions

* 2026-09-18: The accessible-name override is bound together with `role="img"` (A3) — ARIA 1.2 prohibits naming a generic `<span>`.

## Dev notes & deviations

* Icon colour comes from a wrapper layer in the Figma master; don't replicate it — apply the variant's ink token to the icon directly.
* The `content` axis adds and removes layers rather than toggling visibility; render conditionally.
* 2026-10-06: Code differs from the Figma set, which is authoritative: the filled neutral fills and inks, every outline border (it takes the ink, so `currentColor`), and the 20-character truncation (C1).

---

# 4 · Documentation guidance

## Usage

**When to use:**

* Counts, statuses and short labels that should catch the eye — unread messages, new alerts, document states

**When not to use:**

* For a label the user can click to filter or navigate — use Chip Tag
* For a status dot with no label — use Status Indicator
* For a message longer than a few words — use body text, or Banner
* As something the user activates — use Button or Link

**Patterns:**

* Status colours carry status only — success for done, error for action needed. Accents suit categories such as topic tags; neutrals suit counts and metadata.
* Filled draws attention (unread counts, errors); outline suits quieter labels and categories.
* Size l by default; sm in dense places — table cells, list rows, card metadata. Consumers may switch size by breakpoint.
* The consumer places the badge — a count over an icon button or avatar is positioned by that parent.
* One or two badges per item; for a set of labels use Chip Tag.
* A neutral badge with a coloured dot gives a status its own wording (“Live”, “Offline”) without a coloured badge.

**Content:**

* Keep labels to 1–3 characters where possible (“3”, “New”); 12–15 is a practical ceiling (“Pending”, “Archived”). Beyond 20 characters the label truncates (C1).
* The label states the meaning; colour only reinforces it — “Approved”, not a bare green badge.
* Abbreviate large counts (“999+”, “1.2K”), and hide a count badge at zero rather than showing “0”.
* A status indicator repeats the label's status (A5); never use it for a status the label does not state — a green dot beside “Server 3” communicates through colour alone, and a badge cannot show the Status Indicator's tooltip.
* An accessible-name override replaces the visible label in the announcement, so it must contain it: “Page status: Draft”, not “Page status” (A3).

## Gotchas

* Wrapping a badge in a link or button makes the wrapper the control; the wrapper needs the accessible name (I2).
* A dot on its matching filled status badge, such as Positive on filled success, shows only through its border — pair a dot with a neutral or outline badge (F12).

---
