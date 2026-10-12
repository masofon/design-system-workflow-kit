# Accessibility rules

The global rules every component meets, whatever it is. CRDs never restate these: a CRD's
accessibility section holds only what is true of *that* component and no other
(`crd_how_to` § What accessibility belongs in a CRD). Build, review and dev check every component
against this file.

**Starter set.** These rules follow WCAG 2.2 level AA, the config § Accessibility → `standard`.
Keep them, tighten them, or add your own: a changed rule here applies to every component at once.

## Colour and contrast

- **Text contrast**: at least 4.5:1 against its background; 3:1 for large text (24px, or 18.66px
  bold, and up).
- **Non-text contrast**: at least 3:1 for anything needed to understand or operate the UI: input
  borders, focus indicators, icons that carry meaning, chart elements, and the states of
  controls (a checkbox's checked mark, a switch's on track).
- **Check every mode that ships**: config § Accessibility → `contrast_must_pass`. A pairing that
  passes in light and fails in dark fails.
- **Never colour alone.** Status, errors, selection and links carry a second cue: an icon, text,
  a shape or an underline.
- **Disabled controls are exempt** from contrast, but must still read as disabled, and the
  reason a control is disabled should be findable.

## Keyboard

- **Everything works by keyboard**: every interactive element is reachable with Tab and
  operable with the expected keys (Enter or Space for buttons, arrows within composite widgets
  such as tabs, menus and radio groups, Escape to dismiss).
- **Focus order follows reading order.** No positive `tabindex`.
- **No keyboard traps.** Focus can always leave a component, except a modal dialog, which holds
  focus until it closes.
- **Focus is always visible**: a clear indicator, at least 3:1 against its surroundings, never
  removed without a replacement. Focus is never hidden behind sticky headers or overlays.
- **Focus moves deliberately**: into a dialog when it opens, back to its trigger when it closes,
  and never jumps on its own when content updates.

## Pointer and touch

- **Target size**: at least 24×24px for every interactive target, or enough spacing that a 24px
  circle around each doesn't overlap another. Aim for 44×44px on touch.
- **No drag-only actions.** Anything done by dragging has a single-pointer alternative.
- **Actions fire on release**, not on press, so a slip can be cancelled.
- **Hover-only content is reachable without hover**: tooltips also appear on focus, can be
  dismissed with Escape, and stay while the pointer moves onto them.

## Names, roles and semantics

- **Native elements first.** A button is a `<button>`, a link is an `<a>`. ARIA is for what
  HTML can't express, and no ARIA is better than wrong ARIA.
- **Every interactive element has an accessible name**, and the name contains its visible
  label.
- **Icon-only controls** get a text name. Decorative icons and images are hidden from assistive
  technology; meaningful images get alternative text.
- **States are exposed**: expanded, selected, checked, pressed, current, invalid, busy.
- **Structure is real**: headings in order, lists as lists, landmarks for page regions,
  tables for tabular data only.

## Forms

- **Every field has a visible label**, programmatically tied to it. Placeholder text is not a
  label.
- **Required fields** are marked in text, not only with colour or an asterisk without
  explanation.
- **Errors** are identified in text, tied to their field, and say how to fix them. Don't clear
  what the person typed.
- **Instructions and hints** are tied to their field so they're announced with it.
- **Don't ask twice** for information already given in the same process.

## Content changes

- **Announce what changes without a page load**: toasts, inline validation, loading and
  results counts use a live region. Polite by default; assertive only for errors that block.
- **Nothing times out silently.** A time limit can be turned off, adjusted or extended.
- **Status messages don't steal focus.**

## Motion

- **Respect reduced motion.** With `prefers-reduced-motion` set, non-essential animation stops
  or becomes a simple fade.
- **Anything that moves for more than 5 seconds can be paused.**
- **Nothing flashes** more than 3 times a second.

## Text and layout

- **Text resizes to 200%** without loss of content or function.
- **Reflows at 320px wide** with no horizontal scrolling, except content that needs two
  dimensions (data tables, maps).
- **Text spacing can be increased** (line height 1.5, paragraph spacing 2×, letter spacing
  0.12em, word spacing 0.16em) without clipping or overlap.
- **Orientation isn't locked** to portrait or landscape.
- **Right-to-left works**: layout mirrors with logical properties (`inline-start`, not `left`),
  and directional icons flip while others don't.

## Language

- **The page language is set**, and passages in another language are marked.
- **Component-owned text** (a default label, an announcement) is translatable, never hard-coded.
