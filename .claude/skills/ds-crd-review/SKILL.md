---
name: ds-crd-review
description: Review a design system CRD as a document, before or during drafting — accuracy against the token architecture, the accessibility boundary, atom-vs-consumer semantics, Figma-expressibility honesty, front matter hygiene and completeness against the template, and the Figma page's demos against the CRD once design exists. Produces concerns-only feedback for the design team. Use when asked to review, critique, sanity-check or give feedback on a CRD, or to re-review one after the author's changes. Not for writing one (ds-crd-write), editing one (ds-crd-update), or reviewing code.
---

# Review a CRD

Reviews a CRD **as a document**: factual accuracy, internal consistency, house doctrine, and,
once design exists, whether the Figma page's demos agree with it. Your output is concerns only.
You don't edit the CRD.

## Before reviewing

**No repo host CLI or no Figma access?** Do a document-only review: skip the PR re-read and the
Figma checks, and say so in the verdict line, so nobody takes it for a full review.

1. **Read the whole CRD**, front matter included.
2. **Check the format.** `format: 2` in front matter means the current template. Absent means
   an older shape. **Review an older CRD against its own shape.** Absent template sections aren't
   findings — conversion is `ds-crd-update`'s job, on touch.
3. **Read the config** (`.claude/design-system.config.md`): which optional sections and features
   this system uses. A section the config switches off is never a finding.
4. **Verify claims against sources, not memory:**
   - Token names — the published library via `search_design_system` (`design_foundations`
     § Finding tokens). An exported token file lags publishing and may be keyed differently, so
     treat a miss there as unconfirmed, not absent
   - Component and prop claims — config § Repo → `component_code_dir`
   - Sibling claims — the referenced CRDs themselves
5. **Read config § Paths → `crd_how_to`** — it owns the conventions you are reviewing against,
   including the accessibility boundary and the ready-for-dev gate.
6. **Read config § Paths → `design_foundations`** for check 2.

## What to check

### 1. Front matter hygiene

- `status:` flags match reality — drafted requirements are not `not_started`
- `lifecycle:` is right for where the component actually is, and any `stable:` block is complete
- `tier`, `composition` and `category` are present and valid per config § Paths →
  `classification`
- `related_crds` is populated and **reciprocal** — a dependency or precedent named here should
  appear in the other CRD too
- Resolved open questions are removed, and the decision recorded in Design decisions
- Version semantics per `crd_how_to` § Versioning. **There is no changelog section** —
  git holds the diff, the decision logs hold the why. Never ask for a changelog row

### 2. Token & architecture accuracy

Check claims against `design_foundations`, not against memory or another system's conventions.

- Only tokens from the tier components bind (§ Token tiers); verify cited names exist.
  Dimensions reference tokens, not raw px
- **Don't demand token-mapping tables.** Mapping is the designer's design-spec work. Review the
  accuracy of claims made, not the absence of claims not yet due
- Every rule in § System rules holds (colour families, what may vary per brand, whatever the
  system defines). Deprecated tokens (config § Tokens → `deprecated_name`) are a finding
  wherever a CRD reaches for them
- Dark-mode claims respect the theme philosophy in § Mode axes
- Contrast is validated per THEME, not per variant, and brand colours cap what is promisable.

### 3. Atom vs consumer semantics

- Atoms expose bounded VISUAL variants; domain meaning belongs to consuming components. Flag
  domain enums baked into atoms
- Positioning is owned by the parent — a child cannot spec its placement relative to a parent
  it does not know about
- Name components and their permitted variants explicitly. Never an umbrella term: "a CTA
  Button [permitted variants…] or a Link [permitted props…]", not "a CTA"
- Slots state whether they are constrained or deliberately unconstrained, and the default
- Flag orphaned props whose bound structures no longer exist

### 4. Accessibility — against the boundary, not against WCAG in general

`crd_how_to` § What accessibility belongs in a CRD defines what belongs here. Review against
that.

- **A global standard restated anywhere is a finding** — in `B` or `S` statements as well as `A`
  (Tab order, respecting reduced motion, RTL mirroring, focus indicators). What this
  component's motion does under reduced motion is component-specific, not a restatement
- **Do not raise a missing global standard as a finding.** Contrast ratios, keyboard
  conventions, semantic HTML, RTL, reduced motion and touch targets are owned by the general
  accessibility rules (config § Accessibility → `rules`), and CRDs deliberately omit them
- **Do raise a missing component-specific fact**: the ARIA attribute and value this component
  sets, what it announces and how politely, focus order, focus management on state change,
  which parts are decorative, and what its motion does under reduced motion
- **A content swap with no named live-region owner is a finding** — usually the consumer's
  container, present before the change
- **An `A` statement that is only a principle is a finding.** "Must meet WCAG AA" is true of
  everything, so it is not a requirement
- WCAG numbers: **1.4.3** text contrast, **1.4.11** non-text, **1.4.4 is Resize Text**
- Hover-only affordances need a touch answer or an explicitly accepted gap. An unstated gap is
  a concern; a documented one is not

### 5. Figma-expressibility honesty

- Behaviours Figma cannot express — viewport-relative caps, grow-then-scroll, container-derived
  constraints — must be marked code-only, with the Figma approximation stated
- Motion is described in words (what moves, the trigger, reduced motion). The values live in
  the Figma animations in the motion frames. Any duration or easing in the CRD is a finding

### 6. Completeness against the template

- Sections match `crd_template`, minus any optional section the config switches off. A CRD that
  follows another CRD's shape rather than the template is a finding, however tidy it looks
- Statements are atomic, ID'd (`F`/`S`/`B`/`C`/`A`/`I`) and use **anatomy nouns and no others**.
  A statement naming a part the Anatomy does not list is a finding
- **ID gaps** (`crd_how_to` § The requirements contract): a gap in an unbuilt component is a
  finding — renumber; a renumbered prefix in a built one is a **critical** finding — it breaks
  the test names
- `[open]` on anything provisional, `[review]` on anything not machine-testable. An unmarked
  hedge is a finding; so is a stale marker on something design has since settled. Section 4 is
  un-ID'd advice and carries neither
- `C` statements are only copy the component owns or enforces. Consumer copy advice in section 1
  is a finding — it belongs in section 4 under Content
- An `I` statement restating an `F` statement is a finding — it should cite the ID; so is a `C`
  that only restates an `F`
- A statement or decision naming a text style, px value or token is a finding — Figma holds
  those. So is a house term nobody can look up (`crd_how_to`
  § Keeping it tight)
- Figma-only axes and props (a Figma-only breakpoint, say) have a Properties row saying so
- A section 4 line restating a section 1 statement without citing its ID is a finding; so is a
  When-not line that repeats Scope instead of naming the alternative. Dev-facing lines in section 4
  are not — it serves docs generation and the usage agent alike
- Leftover template stubs (`* YYYY-MM-DD:`, an empty `* C1:`) are housekeeping
- Ruled out carries the rejections, each with its why. Empty Ruled out on a CRD full of
  decisions usually means they were not captured
- Open questions have an owner and a By. A row with neither is not a question
- **Code properties and Figma/code mapping do not exist in the template** — the dev team owns
  the API and the docs site documents it. Never ask for them
- Section 4 serves docs consumers after ship. Incomplete is fine pre-design and a finding
  post-ship; empty is never a reason to delete it

### 7. Boundaries & siblings

- Related components consistent with those CRDs; overlapping purposes resolved with explicit
  "use X for Y" guidance
- Every out-of-scope line names the component that handles that case instead

### 8. Tight prose

Against `crd_how_to` § Keeping it tight. Flag, quoting the line:

- A Design decisions or Ruled out entry longer than one line
- A rejection argued in both Ruled out and Design decisions
- A decision that restates a statement instead of citing its ID
- Process in the CRD — build steps, publish state, Figma dates, draft history, tooling
- Doctrine explained rather than named

### 9. Ready-for-dev honesty

Ready for dev is **computed, never stored** — requirements and design specs complete, the
design review held, zero `[open]` markers in section 1, every open question closed or explicitly
deferred. If a CRD's front matter implies readiness the document does not support, say so and
name what is missing.

### 10. Figma demos — once the component page exists

Often where the best findings are. Load `figma:figma-use`, then read the component page
read-only (the CRD's `figma:` link). Compare its demo frames, usage frame included, against
section 1 and section 4:

- A demo showing something a statement forbids, or a "when not to use" case presented as normal
- A statement, state or variant no demo shows, where a demo would help someone use it
- Usage guidance the demos contradict, or demos doing something the guidance never mentions
- Captions that disagree with what the specimen shows

A finding here names the frame or node, and says which side looks wrong: the demo or the CRD.

## Follow-up rounds

**Before every follow-up round, re-read the PR**: its latest commits and the whole thread. Drop a
comment the author has already addressed, and reword one they've partly addressed so it asks only
for what's left. A re-raised fixed comment costs the author trust in the rest.

**Re-review mode**: when asked to re-review, or when the PR already has your earlier review:

1. **Check every earlier comment against the current CRD.** Mark each **implemented**,
   **partly** (say what's left), **not**, or **disputed** (the author replied with a reason;
   quote it, and weigh it rather than repeating the comment).
2. **Run the Figma page checks** if the page exists: the clip sweep (`ds-design-component`
   § 7), detached copies (a frame or group standing in for an instance of the component), and
   bindings against the CRD's wording (a statement about an error border has an error-family
   binding, not a neutral one).
3. **Then § What to check**, on what changed since the last round only.

Report the earlier comments' status first, then new concerns.

## Output contract (strict)

- Lead with a one-line overall verdict
- Then **concerns only**, grouped: Critical fixes → Accuracy issues → Gaps → Housekeeping
- List form; every item actionable; quote the exact CRD line you are referring to
- **No praise paragraphs**, no summaries of what is fine, no restating the document
- **Respect deliberate decisions.** Parked items, accepted gaps and "TBD in Design" markers are
  not concerns — never re-raise them
- When a claim cannot be verified against a source, mark it **"verify"**, not "wrong"
- **Figma findings are drafted for a person to paste.** An agent can't post Figma comments
  (`design_foundations` § Working in Figma via MCP), so give the node link and the comment text

## Cross-domain

| Situation | Read |
|---|---|
| Token claims need verification | `design_foundations`, then the published library |
| Accessibility sections under review | `crd_how_to` § What accessibility belongs in a CRD, plus config § Accessibility → `rules` |
| Figma build questions | `ds-design-component` |

## Self-check before sending

- [ ] Every concern is actionable and anchored to a quoted line or a verified source
- [ ] No concern re-raises a documented deliberate decision
- [ ] No concern asks for a changelog, code-properties table or Figma/code mapping
- [ ] No concern asks for a global accessibility standard that lives in the general rules
- [ ] Unverifiable claims marked "verify", not asserted wrong
- [ ] An older-format CRD was reviewed against its own shape
- [ ] No concern about a section or feature the config switches off
- [ ] Output is concerns-only, grouped, list-form, verdict first
- [ ] On a follow-up round: the PR re-read, nothing already addressed raised again
