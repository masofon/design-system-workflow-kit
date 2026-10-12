---
name: ds-crd-update
description: Edit an existing design system CRD — amend a shipped component, convert an older-format CRD or spec to the current template on touch, reconcile a CRD against what was actually built in Figma or code, and maintain front matter, lifecycle and related_crds. Use whenever a CRD file needs changing for any reason other than writing a brand-new one. Not for reviewing a draft (ds-crd-review) or for reading a CRD to implement against it (ds-crd-read).
---

# Update a CRD — amend, convert, reconcile

Three jobs, one skill, because they interleave: an amendment to an older-format CRD triggers a
conversion, and a build triggers a reconciliation that may close amendment statements.

**Boundary.** `ds-crd-read` locates and applies a CRD during implementation; `ds-crd-review`
critiques a draft. **This skill is the only thing that writes to an existing CRD.**

**Never edit the skills or their reference docs from here**: `.claude/skills/`, the config,
`crd-how-to.md`, `crd-template.md`. They belong to config § Roles → `skill_owners` and change in
their own PRs; an edit inside a CRD PR collides with theirs and slips past review. If a rule looks
missing or wrong, tell the person, propose the wording, and ask them to raise it with
`skill_owners`.

## 0. Ground truth — read before acting

CRDs live in config § Paths → `crd_dir`, **one flat folder, no subfolders**. For anything this
skill doesn't spell out, read the relevant reference rather than working from memory:

| Document | Owns |
|---|---|
| `crd_how_to` | Section-by-section conventions, versioning, the amendment lifecycle, `[open]` markers, the ready-for-dev gate |
| `crd_template` | The target shape |

Don't restate their content in a CRD, a commit message or this skill. One home per fact.

**Format detection**: `format: 2` in front matter means the current template; absent means an
older shape. Both are readable — interpret prose, don't parse. Always **write** the template's
shape.

**CRD means Component *Reference* Document**: most of it is read after the requirements are
settled.

## 1. Decide which job you are doing

**Whichever job it is, the § 3 rule applies: classify every difference, then confirm before
editing.** It bites hardest in reconciliation, but conversion surfaces Figma-vs-code divergences
just as often, and an amendment can invalidate a statement nobody meant to touch.

**Amendment** — new or changed requirements for a component (if config § CRDs → `amendments`
is Yes). Follow `crd_how_to` § Amending a shipped component: a transient block after the
`# CRD: <Name>` title and its horizontal rule, immediately above `# 1 · Requirements`;
statements as `[open]`, reusing IDs where they replace and new IDs where they add; status flipped
to `in_progress`. The block is deleted when design closes it; it isn't a permanent record.

**Conversion** — an older-format CRD, or a spec from another tool, reaching the substance gate.
Map its content onto `crd_template` section by section, using `crd_how_to` § Section by section
to decide where each piece belongs.

**Reconciliation** — the component was built or changed and the CRD has gone stale. § 3.

### The substance gate

An older-format CRD converts **on touch, never in bulk**. A real amendment triggers it; a typo,
a link fix or a changelog line doesn't.

* **Only the CRD in scope converts.** Touching another CRD for a back-link or link fix never
  converts it — make that one edit and leave the rest.
* **Conversion is agreed, never automatic.** Tell the person the CRD is in an older format and
  get a yes first. Convert paired with them: interview them on anything open, unclear or missing
  rather than resolving it yourself. If they say no, make only the edit in hand.

When conversion goes ahead:

* **Two commits.** Conversion first, amendment second, so the amendment stays reviewable instead
  of hiding inside a wall of moved sections.
* **`format:` and `version:` are orthogonal.** Conversion sets `format: 2` and leaves
  `version:` alone — the requirements didn't change. The amendment then bumps `version:` on its
  own terms.
* **Never invent.** Content with no source in the old document is either derived from a real
  artefact (the component's Figma set, the shipped code) or left empty with an honest status
  flag. Empty is information; guessed content reads as approved requirement.
* **Anatomy is load-bearing** and older documents rarely have it. Derive it from the build where
  one exists. With neither build nor design, conversion escalates into an authoring interview
  with the designer — a legitimate path, not a blocker.
* **Strip export artefacts** — inline-styled HTML tables, stray escapes, decorative rules. Plain
  Markdown only.
* Cross-component content — token wiring, global accessibility standards, naming conventions —
  doesn't belong in a CRD. Strip it rather than carrying it across.
* **Never drop an open question silently.** Carry every one across. If an amendment subsumes
  one, the amendment says which and why. A question that disappears looks answered.
* **Accessibility: nothing is deleted on conversion.** The boundary is owned by `crd_how_to`
  § What accessibility belongs in a CRD — read it, don't re-derive it. Component-specific
  statements get `A` IDs; everything else drops into the labelled "carried from the previous
  format, not verified" block below them. Only ID'd statements are the contract. A line that
  fails the test but that the general rules **don't** cover is a gap in those rules: tell the
  person and propose the edit, rather than leaving it parked.
* **Never strip accessibility content by heading match.** Read what's underneath first. Deleting
  a real requirement leaves no trace, and nobody notices until a component ships without it.
* **Distrust derived front-matter values.** A status flag inherited from an older format or set
  by a migration script is a claim, not a fact — check it against the content. A migration that
  matched a *heading* over template placeholder will mark an empty section complete.
* **Deriving contrast pairings from code: check which selector uses which token.** Don't infer
  from the token's name — one whose name reads like link text can turn out to be the
  current-page colour. Where a child component supplies the colour, point at that component's
  CRD instead of duplicating its pairings.

### Cutting, merging or moving a statement

Rule and reason: `crd_how_to` § The requirements contract — Gaps in the numbering. To apply it:

1. **Is it built?** Yes if any of: `lifecycle: built` or `stable` (a stable implementation has
   code and may have tests), a `docs_site:` link, or the component's folder under config § Repo
   → `component_code_dir`.
2. **Not built — renumber.** Close the gap in that prefix, in order. In the same commit, rewrite
   every citation of a moved ID: Design and Implementation decisions, sections 4 and 5 where
   used, other statements, and any other CRD citing this one's IDs (search the folder for the
   CRD's name next to an ID).
3. **Built — retire.** Delete the statement, leave its number unused, never reuse it, and log
   why in Design decisions.
4. **Check.** Search the file for every old ID; nothing may still point at a number that moved.

## 2. Amendment specifics

Scope discipline during design, from `crd_how_to`:

* Statements the amendment names get built.
* Adjustments forced by the change (padding, optical corrections) are made and reported in chat,
  never written into the CRD — they live in Figma and the tokens.
* If the change makes an existing statement false, correct it with a decision entry **and say
  so**. A "small" amendment invalidating several statements probably wanted to be a bigger one.
* Anything neither named nor forced is a future amendment, not this session's work.

Uncertainties go one of two ways, never both: a provisional statement gets an inline `[open]`
marker; a genuine decision needing a named owner and a deadline goes in the Open questions table.

**Owner convention.** A question with no owner isn't a question. Use a person where known, a role
(`Design`, `Dev`) where the work is assigned but the individual isn't, and `[needs owner]` only
when it genuinely has none — and say so in your report.

**A question the build has already answered isn't open.** Delete the row and log the answer as a
dated Design or Implementation decision. Leave it open only if the shipped answer is itself under
review.

**Closing design and folding are one pass.** When an amendment's design closes — usually handed
here by `ds-design-component` § 8 — fold it in the same session, per `crd_how_to` § Amending a
shipped component:

1. **Pre-fold sweep.** Any design decision or answered open question not yet written back? Write
   those first; decisions made after the amendment's last CRD commit are the ones that strand.
2. **Fold.** Rewrite section 1, delete superseded lines and the amendment block, add rejections
   to Ruled out, write Migration if consumers must act.
3. **Leave nothing open.** Resolve **every** remaining `[open]` marker in section 1, not just the
   amendment's, and check any carried-over accessibility block against the general accessibility
   rules, deleting what they cover.
4. **Version** per `crd_how_to` § Versioning: major on 1.x; on 0.x, 1.0.0 only if the contract is
   now complete, otherwise minor.

The design and fold work items can share one PR carrying both ticket refs.

**Every decision and Ruled out entry you write** follows `crd_how_to` § Keeping it tight,
including entries routed here from `ds-design-component`.

## 3. Reconciliation specifics

**Gather both sides programmatically, never from memory.**

* **Figma**: the component's set in the library — variant axes and options, props, bound tokens
  and resolved values, sizes per breakpoint, forced settings on consumed components, Dev Mode
  annotations.
* **Code**: props interface and types, design tokens consumed, behaviours (focus, keyboard,
  responsive).

**Classify every difference, then confirm before editing:**

* **Build drift** — the build deviates, the CRD is right. The fix belongs in the build. Flag it;
  don't paper over it.
* **Design evolution** — a deliberate decision. Update the CRD and log the rationale in Design or
  Implementation decisions.
* **CRD gap** — built but never spec'd. Add it.

Present the diff concerns-first, quoting exact lines, and get the person's call on anything
ambiguous **before** touching the file. Never silently make the CRD ratify a mistake.

## 4. Front matter is machine-owned

Everything in the block is maintained by skills, not by hand. This skill writes:

* `status:` keys as work closes them, and `last_updated:` on every edit. If config § Figma → `context_card`
  is set, update the component page's context card Status to match when `status:` changes
  (`ds-design-component` § 1), or say it needs doing
* `version:` per `crd_how_to` § Versioning: **once per PR**, at the highest level the PR reached;
  sections 4–6 bump only when one reaches complete
* `lifecycle:` — `backlog` → `active` when design starts; `built` when the component ships and
  the `docs_site:` link is written
* `figma:` and `docs_site:` when those links become real
* `tier:`, `composition:`, `category:` — add them to any CRD that lacks them, whatever the edit.
  Values and rules: config § Paths → `classification`. Propose, then confirm
* `related_crds:` — **maintained in both directions.** A back-link is front-matter maintenance,
  not a requirements change: update the other CRD's `last_updated`, never its `version`.
  **Resolve CRDs by front matter, not grep:** map each id in this CRD's `related_crds` to its
  file through `id:` and `name:` (filenames have aliases), and find CRDs pointing here by
  searching every CRD's `related_crds` for this `id`; a body grep misses aliases and hits passing
  mentions. **Append, never replace** the other CRD's lists. A dependency adds this component to
  the other's "used in"; a precedent adds this one as a precedent there. Report which files you
  touched.

### Stable implementations

Only if config § CRDs → `stable_components` is Yes. When a component ships as a **stable**
implementation ahead of a designed core one, this skill records it. Two things change together:

* `lifecycle` → `stable` (from `requested` or `backlog`; a stable implementation is un-designed,
  so it comes before `active`)
* the `stable:` block gains `location` (where the implementation lives) and `status: live`

When a designer picks it up to harden it, `lifecycle` → `active`. When the core component
supersedes it, `lifecycle` → `built`, the `docs_site:` link is written, and the `stable:` block's
`status` becomes `in-deprecation`, with a `retire_by` date once agreed. **The CRD doesn't go to
`deprecated`** — only that implementation retires. Vocabulary and transitions: `crd_how_to`
§ Lifecycle.

**If you are a build skill, don't write this front matter yourself.** Hand over the facts —
component name, that a stable implementation exists, where it lives — and let this skill make the
change. Two writers produce two conventions, and the drift is invisible until something is built
from the wrong side of it.

**Three transitions are human-only** because they are decisions, not consequences: `requested` →
`backlog`, `requested` → `rejected`, and `built` → `deprecated`. Ask; never set them yourself.

**Never store ready-for-dev.** It is computed — `crd_how_to` § The ready-for-dev gate.

## 5. Ship — branch, PR, ticket

**Before every commit** — the first and every fix-up round — **run `ds-simplify-prose`** on the
lines this session changed, and check section 6: anything this session learned that is advice
rather than contract (an `[open]` moved out of section 1, a gotcha found while reconciling) goes
there. Write only what the work produced. Then run `ds-crd-write` § Phase 5's checks for values,
undefined terms and borrowed statements over the same lines.

**Wrap-up** lists anything agreed but not applied, and why (another PR, waiting on someone). An
agreed edit that isn't listed reads as done.

* **One CRD, one ticket, one branch, one PR** — unless the changes are inextricably linked, such
  as a dependency whose contract must change with this one; then they share the PR and its
  description says why. Relationship back-links always ride along. Anything unrelated is its own
  PR.
* **Check the branch before every commit** (`git branch --show-current`). If it isn't this CRD's
  branch, or already carries unrelated work, stop and ask.
* **Get the ticket first** if config § Tracker → `pr_needs_ticket` is Yes. Create or find it with
  `ds-ticket` before the first commit, so the ticket ref (config § Tracker → `ticket_ref`) can go
  in the commit message and the PR never opens without one.

Every change goes through a pull request unless config § Repo → `direct_push` says otherwise.
`<integration>` is config § Repo → `integration_branch`:

```
git -C <repo> checkout <integration> && git -C <repo> pull --ff-only
git -C <repo> checkout -b docs/<name>-crd-<change>
# edit the CRD
git -C <repo> add <crd_dir>/<Name>_CRD.md   # CRD file(s) ONLY
git -C <repo> commit -m "docs(crd): …"       # message format per config § Repo → commit_format
git -C <repo> push -u origin docs/<name>-crd-<change>
```

Then raise the PR with the ticket ref in the description. `ds-ticket` § Linking an item to a PR
covers where else the ref goes and what the tracker links by itself.

* **Follow config § Repo → `commit_format`** and the repo's commit hooks. Never bypass a hook with
  `--no-verify`; if one fails, fix the message.
* **Stage CRD files explicitly.** Never `git add -A`.
* A change spanning a CRD **and** anything else can share one PR, but say so in the description —
  a reviewer of a CRD PR won't expect a skill change inside it.
* If config § Figma → `context_card` is set, mirror substantive changes on the component page's
  context-card changelog — **one line,
  dated** (`ds-design-component` § 1 sets the bar). Rationale stays in the CRD's decision log;
  the card gets the what, not the why.

**After the PR is open:**

* **Move the ticket to `in_review`** (`ds-ticket` § Keeping stages in sync with PRs).
* **Return the clone to the integration branch.** A clone left on the pushed branch catches the
  next session's commits.
* **Every later commit rewrites the PR body** (your repo host's CLI, e.g. `gh pr edit <n>
  --body-file …`) to match what's in the PR — version, open questions, files. Reviewers read the
  body, not the commits.
* **On merge:** ticket → `verify`, then pull the integration branch.
* **Refresh the sibling items' descriptions on merge.** Design and develop items raised before
  the CRD existed say so ("no CRD yet"). When a PR merges that writes the CRD, or changes what
  they point to, rewrite those descriptions to point at the merged CRD, via `ds-ticket` and
  within its rule 1. A stale "CRD not written" sends the next person looking for nothing.

## When to use

* "Add X to the Y CRD" / "the CRD needs amending for …"
* Post-build wrap-up after `ds-design-component`
* A component changed in code or Figma and the CRD has gone stale
* An older-format CRD needs bringing to the current template because something substantive is
  changing in it
