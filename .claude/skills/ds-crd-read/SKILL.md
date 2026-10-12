---
name: ds-crd-read
description: Find and apply a component's CRD (Component Reference Document) when implementing or reviewing it in code. Use when implementing or reviewing a component by name, locating a CRD, or checking code against the approved spec in the CRD folder. Not for writing a CRD (ds-crd-write), editing one (ds-crd-update) or critiquing one (ds-crd-review).
---

# Read a CRD

The CRD is the contract for a component. Find it, read it, and implement or review against it.

## Location

Config § Paths → `crd_dir`: one flat folder. State is the `lifecycle` key in each CRD's front
matter, not the folder. The values and their meanings are in `crd_how_to` § Lifecycle.

## Name → filename

The filename follows config § Paths → `crd_filename`. With the default `{Name}_CRD.md`,
PascalCase becomes underscores: `BasicCard` → `Basic_Card_CRD.md`.

**Don't rely on the filename.** Glob `crd_dir` for the name, match loosely (case, underscores,
spaces, a missing suffix), and confirm against each candidate's `name:` and `id:` front
matter. The `id` is the stable handle. If more than one CRD plausibly matches, show the
candidates and ask.

## Reading it

- **Section 1 is the contract**: the numbered statements (F, S, B, C, A, I) are what the code
  must satisfy. Cite them by ID in reviews and tests.
- **An `# Amendment` block above section 1** is a change in flight. It is not yet the
  contract: build against the folded section 1 unless you were asked to build the amendment.
- **`[open]` statements** are provisional. Flag any you're relying on.
- **Lifecycle tells you whether the component has been designed.** `requested` or `backlog`:
  nothing is designed yet. `active`: design is in progress, so check with the designer before
  building. `stable`: implemented ahead of design. `built`: designed and shipped — code and CRD
  should match, and a mismatch is a finding. `deprecated` or `rejected`: don't build against it.

## When to use

- Someone names a component to implement or review ("implement BasicCard")
- Validating code against its approved spec
