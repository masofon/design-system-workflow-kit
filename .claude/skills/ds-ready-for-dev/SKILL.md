---
name: ds-ready-for-dev
description: Check that a design system component, or an amendment to a shipped one, is ready to hand to dev, then hand it over — the CRD gate, the CRD and Figma library checks (each read from the skill that owns the rule), the human checks, PRs and tickets — reporting block and warn findings first, then moving the work items to verify, the container and develop item to ready for dev, and returning the clone to the integration branch. Use when asked whether a component or amendment is ready for dev, to run the ready-for-dev check, to hand a component to dev, or to move a component's ticket to ready for development. Not for writing or fixing a CRD (ds-crd-write, ds-crd-update), building the Figma side (ds-design-component), or reviewing a CRD's prose (ds-crd-review).
---

# Ready for dev

One run before a component or amendment goes to dev: **check, report, stop; close only when
told.**

**This skill owns almost no rules.** Each check names where its rule lives: apply it from
there, not from a summary here, which would drift. This skill owns only the gate, the human
checks, PRs, tickets, publish and the clone.

**The check pass is read-only.** Fix nothing while checking — not the CRD, not Figma, not a
ticket. Fixes go through the skill that owns them (`ds-crd-update` for the CRD,
`ds-design-component` for the library), then re-run the section they touched. Never edit a
skill from here: report a missing or ambiguous rule as a skill gap.

**Respect deliberate decisions.** A designer's documented override (a backdrop bound to a page
token rather than a container token, say) is not a finding. If you can't tell whether something
is deliberate, ask.

## 0. Set up

1. Check out config § Repo → `integration_branch` and pull fast-forward only, so you check
   what's merged, not a stale clone.
2. Find the three things the run is about, and confirm them with the person:
   - the **CRD**, by `name:` front matter (`ds-ticket` § Finding the CRD)
   - the **container** and its work items (`ds-ticket`; the adapter says how to list children)
   - the **library page and component set** (config § Figma → `component_library`), from the
     CRD's `figma:` link
3. **New component or amendment?** An amendment's container follows config § Tracker spin-up →
   `amendment`. Its checks add § 1.3. Skip § 1.3 entirely if config § CRDs → `amendments` is No.
4. Read config § Paths → `crd_how_to` § The ready-for-dev gate. Load `figma:figma-use` before
   any `use_figma` read.

## 1. Check pass

Each finding is **block** (must pass before dev) or **warn** (fix soon; it serves later
consumers and doesn't hold dev). **(human)** checks are questions for the person: never infer
the answer.

### 1.1 The gate — block

The four conditions in `crd_how_to` § The ready-for-dev gate:

- `requirements: complete` and `design_specs: complete`
- **(human)** Design review held — ask who held it, and whether it covered the full § 1 against
  the library set and section 6
- Zero `[open]` markers anywhere in section 1
- Every Open questions row closed, or `deferred: <reason>`

### 1.2 CRD content — block

Run `ds-crd-review` on the CRD. Its findings are this section's: Critical fixes, Accuracy issues
and section 1–3 Gaps are block; Housekeeping is warn. On top of the review:

| Check | Rule lives in |
|---|---|
| No § 1 statement contradicts the library set — variant axes, props, Properties table rows | `crd_how_to` § When Figma & the CRD disagree |
| Every design decision made in Figma is written back — nothing only in chat, a context card or an annotation | `ds-design-component` § 6 |
| ID numbering: no gaps before build; retired, never renumbered, after | `crd_how_to` § The requirements contract |
| `version`, `last_updated`, `lifecycle` right | `crd_how_to` § Versioning, `ds-crd-update` § 4 |
| `related_crds` reciprocal, found by front matter | `ds-crd-update` § 4 |

### 1.3 Amendment only — block

`ds-crd-update` § 2 *Closing design and folding are one pass*, and `crd_how_to` § Amending a
shipped component:

- Amendment folded: no `# Amendment` block left, superseded lines gone
- **(human)** Pre-fold sweep done — any decision or answered question made after the last CRD
  commit written back
- Every `[open]` in section 1 resolved, not just the amendment's
- Rejected options in Ruled out; Migration written if consumers must act
- Version bumped per § Versioning (major; on 0.x, 1.0.0 only if the contract is complete)

### 1.4 CRD later sections — warn

Sections 4–6 against `crd_how_to` § Section by section and the review's completeness lens:
§ 6 Usage filled from an interview and never restating § 1. § 4 only if config § CRDs →
`brand_validation` is Yes: it lists the foreground/background pairings runtime validates, no
measured ratios. § 5 only if config § CRDs → `cms_behaviour` is Yes and the component becomes a
CMS block.

### 1.5 Figma library — block

Read-only `use_figma` passes on the page and set. Check against:

| Area | Rule lives in |
|---|---|
| Page name and symbols, placement, canvas | `ds-design-component` § 1, config § Figma → Library conventions |
| Context card — metadata only, header links, Status matches `status:`, changelog entry for this change | `ds-design-component` § 1 |
| Component description — on the set only, purpose, demo-tile note, CRD link, nothing else | `ds-design-component` § 1 |
| Dev resource link — `get_design_context` on one variant lists the CRD's repo URL under **Documentation**. Missing or wrong: report the exact URL for the person to add; you can't add it | `ds-design-component` § 1 |
| Figma-only axes and props; mode-dependent literals right at every mode | `ds-design-component` § 3 |
| Code-only props | `ds-design-component` § 4 |
| Dev Mode annotations for what Figma can't express, each applied once | `ds-design-component` § 8 |
| Every binding resolves across every mode axis (`design_foundations` § Mode axes); whole-page text style and token fill sweep | `ds-design-component` § 7 |
| Slot and prop-ref parity across variants; default variant top-left, option order, set layout | `ds-design-component` § 3 (Laying out the set) and § 4 |
| Page frames and layout. If config § Figma → `page_layout` is `team`: matches `reference_page` and the Library conventions keys. If `kit`: config `doc_frames` and the kit guidance | `documentation-frames.md` beside `ds-design-component` |

Plus, **(human)**: **the library is published** with this work — dev sees only published
(`design_foundations` § Publish gates). Only config § Roles → `publisher` can publish.

**The ready-for-dev mark (config § Figma → Page status symbols → `ready_for_dev`) is never
yours.** Only config § Roles → `ready_for_dev_awarder` sets it. If everything passes, say the
page is eligible and leave it to them.

### 1.6 PRs and tickets — block

- **Work items present.** New component: every work item in config § Tracker spin-up →
  `component`. Amendment: those in `amendment`, with Write only if one was raised
  (`ds-ticket` § Spinning up an amendment's items).
- **Every PR merged.** Find each work item's PRs by its ticket ref (config § Tracker →
  `ticket_ref`) with your repo host's CLI (`gh pr list --state all --search …` on GitHub), and
  confirm each is merged. An open PR is a block; so is a design or CRD work item with none. Each
  PR body carries the ticket ref and matches what merged.
- **Develop item** carries the CRD link and isn't at stage `blocked`.
- **Item descriptions** current with what was decided and built.
- **Owners.** Read the assignee and creator on the container and every item you'd move
  (`ds-ticket` § Two rules, every time).
- **Clone** on `integration_branch`, clean, and `git worktree list` shows no leftover worktree
  for this work.

## 2. Report, then stop

- One-line verdict: **Ready**, or **Not ready — N blocks**.
- **Block**, then **Warn** — concerns only, each quoting the line or naming the node, with the
  home of the rule it breaks.
- **Human questions** — every (human) check, asked together.
- **Closure plan** — each move in § 3, with the item's owner where it isn't the person running
  this.
- **Not checked**, and why — anything you couldn't reach (Figma unavailable, a PR in another
  repo).
- **Skill gaps** — any check whose rule had no home, or whose home was ambiguous.

Then stop. Re-run any section a fix touched before closure.

## 3. Closure

Only with zero blocks, every human check answered yes, and the person saying go. Moving an item
someone else owns needs the person to name it — the closure plan is how they do. Every move
goes through `ds-ticket`.

1. **Work items to stage `verify`** — CRD, Design, and Fold for an amendment. **One at a time,
   in sequence: confirm its PRs merged, then move it, then the next.** Never move ahead of the
   merge, and never in parallel, so no ticket runs ahead of its PR. Skip an item already at or
   past `verify`.
2. **Container and develop item to stage `ready_for_dev`** (`ds-ticket` § Keeping stages in
   sync with PRs). Dev review is the develop item's first step, not a precondition. A develop
   item already at `in_progress` or later has started — leave it and say so.
3. **Clone back on `integration_branch`**, pulled. Remove a leftover worktree only after
   confirming its branch merged, and ask first.
4. **Hand back every item URL** (config § Tracker → `item_url`), container first.

Apply config § Tracker stages: never change a sprint or iteration, never move an item backward,
and never move one out of the stages listed there. If a tracker write is refused or hangs,
follow the tracker adapter § Failure modes.

## After build — not this skill

Name these as next steps in the report; don't check them:

- Dev review, the develop item's first step, resolved in the CRD before the develop PR merges
  (`crd_how_to` § The ready-for-dev gate)
- Code Connect mapping created, or updated if props changed (`figma:figma-code-connect`)
- CRD reconciled against the build, `docs_site:` set, `lifecycle: built` (`ds-crd-update` § 3)
