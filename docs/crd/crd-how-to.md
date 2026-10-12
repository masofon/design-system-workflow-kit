# How to work with CRDs

A guide for humans. Agents get their instructions from the skills.

## What a CRD is

**CRD stands for Component *Reference* Document**, not *Requirements*. Only the first section is requirements; calling the whole thing a requirements document says it is finished when development starts. It is consulted for the life of the component, not filed away at handover.

It travels with the component: requirements before design, specifications after, implementation decisions before & during dev, and changes after ship. It is not a spec dump & not documentation. Its job is the delta — what Figma, the code & the agent rules cannot tell you: behaviour, accessibility intent, rules, edge cases, decisions & their reasons.

Where Figma, the code, the docs site or the agent rules express something fully, link to it rather than restating it. Where they express it only partly — a property name is in Figma, but the intent behind it is not — the CRD carries what is missing, & may repeat the name it hangs off. One home per fact; the exception is a key you need to hang the missing part on.

## Related CRDs

`related_crds` is a context-loading instruction, not a relationship map. A CRD earns a place there if an agent working on this component needs to read it to do the job correctly. Two reasons:

**Dependency** — children named in the anatomy, & parents that enforce something on this component (Modal dictating its footer button sizes). An agent *must* read these; missing one breaks the build.

**Precedent** — structurally similar components this one should stay consistent with. An agent *should* read these before inventing an approach; missing one produces drift rather than failure.

**Related CRDs are maintained in both directions.** Adding a dependency to this CRD means that component's "used in" gains this one, & its front matter too if it is genuinely constrained by this component. Adding a precedent means that component's front matter gains this one as a precedent, because consistency is symmetric. The asymmetry matters: a child does not need to load its parent to be built, so mirroring every dependency both ways would inflate context for nothing. The skills do this automatically & report which files they touched.

The test: would an agent build this wrong without reading it? If no, it belongs in Integration prose, not the front matter. The "used in" list can be long & non-exhaustive precisely because nobody loads it.

## Lifecycle

`lifecycle` is the CRD's state, at a glance:

* **requested** — proposed, not committed to
* **backlog** — accepted, nobody has started
* **active** — being worked on now
* **stable** — shipped as a stable implementation, ahead of a designed core component. Think
  of it as backlog with a head start: something exists and is usable, but no designer has
  considered it yet. Only if config § CRDs → `stable_components` is Yes
* **built** — shipped in the design system proper; the CRD now serves docs & downstream consumers
* **deprecated** — the component is retired, the CRD kept for history
* **rejected** — proposed & declined. The CRD stays, so the same request cannot arrive again
  without its reasons attached

One track, one CRD:

```
requested -> backlog -> active -> built
                    \-> stable -/
```

`stable` sits **before** `active`. A stable implementation is undesigned by definition; it
becomes `active` when a designer picks it up to harden it, & `built` when that work lands. A
component with no stable implementation goes `backlog` → `active` → `built`.

`active` means **a designer is designing it** — exactly what a stable component lacks. Keeping
it that strict lets "has this been designed?" be answered from the lifecycle alone.

**`stable` describes the CRD, not the winding-down of the code.** Once the hardened component
lands, the CRD is `built`; the *implementation* retires, tracked in its own block:

```
stable:
  location:            # where the stable implementation lives
  status:              # live | in-deprecation
  retire_by:           # YYYY-MM-DD, once a deadline is agreed
```

The deadline governs the code, not the document, so a CI rule never has to read a document's
state to decide whether to block a build.

State never lives in folders. Every CRD sits in one directory: a path is a second home for a fact, & moving a file breaks every link to it.

**`status`** tracks each section separately: `not_started`, `in_progress` or `complete`, one key per section in the template. `lifecycle` never duplicates it — "in design" or "ready for dev" is answerable from the status map & the ready-for-dev condition, & a second copy of a computable fact is how drift starts.

## Who reads what

Every section states its audience at the top. In short:

| Section | Written | By | Consumed by |
|---|---|---|---|
| Amendment | While a change is in flight | Designer/PM | Everyone: it flags the contract is under revision |
| 1 Requirements | Pre-design | Designer | Everyone: it is the contract |
| 2 Design specifications | Post-design | Designer | Devs & build agents |
| 3 Implementation | Pre-dev & during | Developer | Build agents, future maintainers |
| 4 Brand validation *(optional)* | Post-design | Designer | Brand setup & runtime brand checks, not the build |
| 5 CMS behaviour *(optional)* | Post-design | Designer | CMS block work & runtime agents, not the build |
| 6 Documentation guidance | Post-design & post-dev | Designer & dev | Docs generation & usage agents |
| 7 Migration | Only when consumers must act | Whoever changes it | Consumers of the component |

Sections 4 to 6 exist because the CRD stays relevant after the component ships. Agents building products, setting up brands & writing documentation consume them later. If you are building the component, skip them; do not delete them.

**Sections 4 & 5 are optional for the whole system**, switched by config § CRDs → `brand_validation` & `cms_behaviour`. Brand validation is for systems where customers or sub-brands supply their own colours at runtime; CMS behaviour is for components a content editor configures. When the config says No, the section & its `status` key are deleted from every CRD, & nothing below that mentions them applies.

## The requirements contract

Only the ID'd bullets in section 1 make up the contract: F1, S1, B1 & so on. If a line has an ID, it is a requirement; if it does not, it is vocabulary, framing or routing (Purpose, Scope, Anatomy, Ruled out, Open questions, Notes).

That means the contract is not just the subsection called Functional requirements. States, Behaviour, Content, Accessibility & Integration statements are requirements too; those subsections exist because that is how people think & review, not because their contents are softer.

Everything else follows two rules:

**Atomic & testable.** Each bullet states exactly one requirement, phrased so it can be verified true or false. A bullet that could be half-satisfied is two bullets — "a button has a leading or trailing icon, never both, and icons render at the size's icon token" can pass & fail at once, so the audit cannot say which. "A button has a leading OR a trailing icon, never both" passes; "icons should feel balanced" does not. Statements that cannot be machine-tested carry a `[review]` prefix: they are verified in the review & audit pass rather than by tests.

**Numbered.** Per-concern IDs: F for functional, S for states, B for behaviour, C for content, A for accessibility, I for integration. The IDs exist so each requirement can be verified individually: the build maps tests to them & cites the ID in the test name, so a failing test points straight at the statement that demanded it. If it is not written as a statement with an ID, it will not be verified. How the build does that verification is the pipeline's business, not the CRD's — nothing it produces gets written back into this document.

**Gaps in the numbering.** Before the component is built, IDs are only cited inside the CRDs, so when a statement goes, renumber its prefix with no gaps & update every citation in the same commit. Once it is built (`lifecycle` `stable` or `built` — a stable implementation has code too), tests carry the IDs: never renumber. Retire the ID, never reuse it, & log why the statement went in Design decisions.

There is no separate definition-of-done list. Section 1 is the enumerable list — one test per line, each provably done — so the checklist cannot drift from the requirements it checks. Pipeline mechanics, such as setting `docs_site:` after build, belong in your build process, written once for every component.

## Anatomy & Ruled out

Two vocabulary sections that make the rest of the document work.

**Anatomy** names the parts: the named regions of the component, the things you can point at in a design (container, label, leading icon, spinner). Some are sub-components, some are plain elements; Anatomy names both without distinguishing. Properties are not parts — `size` is something you set, `container` is something you point at. Everything hangs off these nouns: requirement statements, token paths, Figma layers, prop names, decisions. Write statements using anatomy terms & no others; if you need a word that is not in the anatomy, the anatomy is incomplete.

**Ruled out** records considered-&-rejected design space ("a third size: no evidenced need"), one line each with the why or a link to the decision. It is not out-of-scope, which routes to other components; its job is preventing re-litigation, & it matters most for agents doing best-practice research, which will otherwise cheerfully propose the option you rejected six months ago. When a decision permanently closes a door, add the line here in the same commit.

## Provisional statements & the [open] marker

Pre-design, the CRD is partly a design brief, and briefs are allowed to contain hunches. "Consider stacking buttons vertically on mobile" is a legitimate prompt for design exploration. The rule is that it must be visibly provisional:

```
[open] B4: Consider stacking buttons vertically on mobile
```

Design closes every marker with one of three exits:

1. **Promote.** It becomes a testable statement: "B4: Buttons stack vertically below the mobile breakpoint." Log it in Design decisions.
2. **Move.** It turned out to be advice, not a requirement. It goes to Documentation guidance as a usage pattern.
3. **Cut.** It did not survive contact with the design. Log it in Design decisions with the why.

Bigger uncertainties that need a named owner & an actual decision go in the Open questions table instead, each with its deadline in the By column ("before auth screens are designed"): timing is often the real content of a question. Do not put the same thing in both places.

## The ready-for-dev gate

"Ready for dev" is a condition, not a status you set. It is true when all of these are:

* `requirements: complete` & `design_specs: complete` in the front matter
* The design review is held — a designer has checked the CRD against the component's set in the Figma library (see below)
* Zero [open] markers anywhere in section 1
* Every Open questions row closed or explicitly marked "deferred: [reason]"

Brand validation, CMS behaviour & docs guidance are not dev blockers; they serve later consumers, so they do not hold the gate.

It is computed rather than stored because a stored flag can be true in the front matter & false in reality the moment anything upstream changes. Anyone — human or agent — can check the condition in seconds, & a build agent verifies it before starting rather than trusting a flag someone set optimistically.

There are two reviews, & neither is a front matter field — a stored flag would only claim a conversation happened.

* **Design review, before ready for dev.** A designer checks the CRD against the component's set in the Figma library: the design covers every statement, intent is captured, every dual option (link or button, badge or chip) is impossible to miss in both places, nothing contradicts the file, & section 6 is sound. It is the one part of the condition nothing else can infer, so a human confirms it when the `ds-ready-for-dev` skill asks.
* **Dev review, during dev.** The developer's first step on the develop work item: is every statement buildable & testable, does anything conflict with the existing API or code? Done against the Figma file, so the CRD-to-Figma join is checked twice. It is part of the build's done condition — the develop PR does not merge until it is held & anything it raised is resolved in the CRD.

If a build agent finds an [open] marker or an open question, it stops & flags rather than guessing. That is deliberate. Guessing is how components get built wrong quietly.

## When Figma & the CRD disagree

Precedence, in order:

1. Figma for visual specification
2. The CRD for behaviour, accessibility, rules & requirements
3. Your code conventions & agent rules, for implementation standards
4. The docs site & code for the built API

If they conflict on the same fact, do not silently pick one. Flag it. A conflict means one of them is stale, & picking the wrong one hides the staleness.

## Developers: your sections

**Dev review comes first** (§ The ready-for-dev gate). Then two habits, both in section 3.

**Implementation decisions, before you build.** If you research best practices, compare how other libraries handle something, or decide on a pattern ("render status as a child of the indicator, not an overlap prop"), capture it here, dated, one line plus a short why. In your prompt only, the decision shapes one build & evaporates; here, it shapes every rebuild, refactor & audit, & nobody re-litigates it. Record reversals explicitly ("reverses the earlier X decision"), & cite precedents from other CRDs by name ("the Divider lesson"). Named precedents build case law: one decision, cited three times, becomes a convention no one has to restate.

**Dev notes & deviations, during the build.** Limitations you hit, quirks, performance notes, & anything you did differently from the design or the requirements. Deviations get flagged to the designer, not just recorded; the CRD is not a place to bury a disagreement.

## Designers: your sections

Section 1 is yours before design; sections 2, 4, 5 & 6 are yours after. The one that needs discipline is **Design decisions**. The bar for logging: could a future person or agent plausibly propose the opposite? If yes, log it with the why. If no — you reworded a statement, you fixed a link — it is a commit message, not a decision. Logging starts with the first decision, not with Figma: an `[open]` closed during the requirements interview is logged then. And if a new entry fully supersedes an old one, say so & delete the old one; never leave two entries where one is now wrong. Every change from the original requirements gets a dated line. If the design changed a requirement, update the statement in section 1 in the same commit as the log entry. Before writing statements back, read each against Purpose: a statement that misses a case Purpose implies is incomplete. A decision log that disagrees with the requirements above it is worse than no log.

For section 2, the Properties table lists every property, so an absent row means the property does not exist. Only the description column does real work: intent, cross-property rules, why a property exists — the things MCP cannot surface. Leave it empty where Figma captures the property fully; an empty description reads as "nothing to add", which is information. Figma-only axes & props (a breakpoint axis that exists only to show a literal at each breakpoint) get a row whose description says "Figma-only — don't build". There is no code properties table & no Figma-to-code mapping on purpose: the development team owns the API, & the built API documents itself on the docs site & in the code.

## Section by section

What goes in each section, & who consumes it. The template is a bare skeleton; the guidance lives here.

**Purpose** — two or three lines: the problem this solves & its primary job. No feature lists.

**Scope** — what this is & is not. Every out-of-scope line points at the correct alternative component, so neither humans nor agents reach for this one wrongly.

**Anatomy** — the named parts (see above). One line each; mark optional parts.

**Functional requirements (F)** — capabilities & rules. Edge cases are requirements & live here, not in a later section. State rules so they are impossible to miss ("F4: a button has a leading OR a trailing icon, never both").

**States (S)** — every state, one line each: interactive (hover, focus, pressed) & application (disabled, loading, error). The state vocabulary is shared across the CRD, Figma, tokens & code — one name per state, everywhere.

**Behaviour (B)** — interactions, keyboard behaviour, responsive behaviour, animation & motion. Figma cannot express this, so be complete here rather than thorough about visual description.

**Content (C)** — only copy the component owns or enforces: fixed or translated strings, truncation at N characters, casing through the text style. If a build test cannot fail on it, it is not a C statement. A C that only restates an F ("Heading required" beside F1) is redundant — the F holds it. Advice to the consumer on what to write goes to section 6, under Content.

**Accessibility (A)** — component-specific only; see § What accessibility belongs in a CRD. Include the ARIA attributes & values this component sets, what it announces & when, its focus order & focus management, & anything unusual to it. Do not restate global standards.

**Integration (I)** — composition rules this component enforces on its children, plus structural rules. Ownership: if a parent enforces something on a child (sizes, placement), that lives in the PARENT's CRD. Children & Used in are linked lists, not requirements. An I statement never restates an F statement; cite its ID.

**Ruled out** — rejected design space (see above).

**Open questions** — owner, By & status per row. Small uncertainties stay inline as `[open]` markers instead.

**Notes** — relevant context that is not a requirement: brand variability expectations, constraints from outside the component. Leave empty rather than pasting boilerplate.

**2 · Design specifications** — the properties table & design decisions (see above). Written post-design.

**3 · Implementation** — implementation decisions before you build, dev notes & deviations during (see above).

**4 · Brand validation** *(only if config § CRDs → `brand_validation` is Yes)* — token pairings needing contrast validation, split by when they can be checked: at brand creation (opaque fills) or at placement (transparent fills needing parent surface context). Check the token's value, not its name: any alpha makes it transparent. Plus exemptions & anything the validation logic needs. Consumed by brand setup tooling, not the component build.

**5 · CMS behaviour** *(only if config § CRDs → `cms_behaviour` is Yes)* — what a self-serve user gets when they add this as a block: layout constraints, the exposed subset of properties, & guardrails protecting them from broken or inaccessible configurations. Consumed by the CMS block work & the runtime agent, not the component build.

**6 · Documentation guidance** — when to use, when not to (pointing at the alternative each time), common patterns, consumer content guidance, & gotchas. This is where `[open]` statements land when they turn out to be advice rather than requirements. It is advice, not contract: no IDs & no `[review]`. Seeded during the requirements interview & filled in a short interview once design is done, so it never waits for a docs pass. Two audiences read it — docs site generation & the designer-facing usage agent — so a line aimed at developers ("pass the real count") belongs as much as one aimed at designers.

Section 6 never restates section 1. A line that repeats a statement cites its ID or goes. When-not names the alternative to use; it is not Scope's out-of-scope line again. A gotcha gives the consequence in plain words, then the ID.

**7 · Migration** — only while a change requires consumer action.

## What accessibility belongs in a CRD

An accessibility statement earns its place only if it is true of **this component and no other**. Everything else already has a home: the general accessibility rules your build & review apply on every component (config § Accessibility → `standard`, plus `rules`).

**The test.** Would this sentence still be true pasted into a different component's CRD? If yes, it is not yours. If no, keep it & give it an `A` ID. It applies to every statement, not just the Accessibility subsection: Tab order, reduced motion & RTL mirroring often turn up as B or S statements.

**Never write these.** All are covered by the general rules:

* Semantic HTML — element choice, link-vs-button, heading hierarchy, list markup
* ARIA mechanics — when to reach for `aria-label`, `describedby`, `labelledby`, `live`, `expanded`, `current`; prefer-native-HTML
* Contrast ratios — 4.5:1 text, 3:1 large text, UI components, non-text & focus indicators
* The disabled-state contrast exemption
* Keyboard conventions — that tab order follows DOM order, Enter/Space activating buttons, Escape closing dialogs, arrows within groups
* Focus indicators — visibility, 2px, 3:1, `:focus-visible` over `:focus`
* Screen reader basics — alt text, label association, announcing dynamic content
* RTL — logical properties, `:dir(rtl)` flips, directional icons
* Reduced motion — `prefers-reduced-motion`
* Touch targets — the 44x44 minimum & the pseudo-element technique
* Modal behaviour — focus trap, focus return, `aria-hidden` on page content
* Loading states — `aria-busy` & `aria-live`
* Form field wiring — `aria-required`, `aria-invalid`, error association
* WCAG criterion numbering — cite from the general rules

**Do write these.** Each names something specific to this component:

* The ARIA attribute it sets & the value it sets — `aria-label="Breadcrumb"`, `aria-current="page"`
* What is announced, when, & how politely. An `aria-live` politeness choice is a design decision
* Which parts are deliberately decorative & excluded from the accessibility tree
* **Focus order within the component**, where it is not simply DOM order, or where a part is deliberately not focusable. Breadcrumb's current page is skipped, & that is a requirement rather than an accident
* **Focus management on state change** — where focus goes after a dismiss, delete, expand, collapse, close or async update. General rules cover the modal case only; every other component's answer is a design decision
* Keyboard behaviour beyond platform convention
* **Who owns the live region** when this component's content swaps in or out — usually the consumer's container, present before the change. A region inserted along with its content is not reliably announced
* Accessibility-driven design decisions — they start as an `A` statement & end up in Design decisions

**No `A` statement may be a principle alone.** Every one names an attribute, a value, an announcement or a behaviour. "Must be accessible" & "must meet WCAG AA" are not requirements; they are true of everything.

**Failing the test does not promote it to the general rules.** Three outcomes: it is already covered, so it goes (the usual case); it is a genuine gap, so raise it against the general rules rather than writing it here; or it is component-specific but written too vaguely, so rewrite it & keep it. The same sentence appearing in three or more CRDs is a prompt to go looking for a missing global rule, not grounds to promote one.

**Converting an older CRD: nothing is deleted.** Statements that fail the test are not removed during conversion. They drop below the ID'd ones into a labelled block:

> _Carried from the previous format, not verified as component-specific. Check against the general accessibility rules & delete once confirmed covered._

Only ID'd statements are the contract, so the block cannot be mistaken for one, & nothing is lost while it waits to be checked. **Read the content under every accessibility heading before moving anything — never match on the heading alone.** An old heading tells you nothing about what sits under it, & real requirements hide under generic-sounding ones: much of what sits under "Focus management" is genuinely component-specific.

## Front matter is not yours

Everything in the front matter block is maintained by the skills, & not by hand. `lifecycle`, `tier`, `composition`, `category`, `status`, `version`, `format`, `last_updated`, `figma`, `docs_site`, `related_crds`, `id` & `name` are all either derivable from the work, governed by config § Paths → `classification`, or need to change in more than one file at once — `related_crds` is maintained in both directions, a status change moves `lifecycle` from backlog to active, closing a section bumps the version. A human editing one of them by hand updates one copy of a fact that has two, & the drift is invisible until something is built from the wrong side of it.

So the writing happens through Claude Code, & you read the result rather than typing it. In practice that means both windows open: Claude Code doing the editing, & the CRD in preview beside it so you can see each change land in context. An occasional hand tweak to a line of prose is fine. Front matter, never.

The exceptions are all one exception: a decision nothing can infer. `requested` to `backlog`,
`requested` to `rejected`, & `built` to `deprecated` are set by a human, because a skill making
them would be claiming a decision it did not see.

Everything else is a consequence of work that already happened, & consequences are the machine's
job — including `backlog` to `stable` when a stable implementation ships, `backlog` or
`stable` to `active` when a designer starts work on it, & `active` to `built` when the component
lands in the design system.

## Versioning

The version in the front matter tracks this document's maturity, at a glance. The skills apply these rules; it is here so you can read a version, not so you can set one. It has no relationship to the components package version: the package releases the whole library at once, so a per-component CRD version could not mirror it even if you wanted it to.

* **Patch** — a change in sections 1 to 3, which the build reads: corrections, clarifications, a decision entry, a closed open question, the `docs_site` link added after build.
* **Minor** — a section reaches complete. The only bump sections 4 to 6 get.
* **1.0.0** — the CRD is complete & the contract is fixed. That is the gate, not the ship date; dev happening afterwards is a patch.
* **Major** — an amendment completes. Each major version is one generation of the component's requirements contract, so 2.0.0 is the contract after the first amendment.
* **Pre-1.0 amendments** — on a 0.x CRD, a completed amendment goes to 1.0.0 only if the fold leaves the contract complete (no markers, no open questions); otherwise it is a minor bump.

**One bump per pull request**, at the highest level it reached — not one per commit. A back-link or front-matter-only edit never bumps.

## Amending a shipped component

Three months after the card ships, it needs significant new functionality. Do not splatter the new requirements through the old contract, & do not keep the superseded ones for reference. Amend first, then rewrite.

**Write the amendment.** A section at the top of the document, above section 1: a sentence or two of context (what is changing & why now), then the new & changed requirements as provisional `[open]` statements, reusing existing IDs where they replace a statement, new IDs where they add one, deletions noted. Nothing more structured than that — it exists for one design cycle & then it goes. It sits at the top because anyone reading the contract needs to know it is under revision before they act on it. Flip the affected `status` keys to `in_progress`.

**Scope during design.** Statements the amendment names get built. Design adjustments forced by the change (padding, spacing, optical corrections) get made & reported in chat, never written into the CRD: they live in Figma & the tokens. If accommodating the change makes an existing contract statement false, that statement gets corrected with a decision entry, & you should hear about it — a "small" amendment invalidating several statements probably wanted to be a bigger amendment. Anything neither named nor forced by the change is a suggestion for a future amendment, not this session's work.

**Design closes it — & the fold happens in the same pass.** Same as any `[open]` marker: every statement is promoted, moved or cut, & logged in Design decisions. The why-now from the amendment's context line lands here too, which is where it lives permanently. Closing design is not a separate step from folding; the design & fold work items can share one pull request.

**Sweep before folding.** Any design decision or answered open question not yet written back goes in first — decisions made after the amendment's last CRD commit are the ones that get stranded.

**Then fold it in.** Statements are rewritten, added, or deleted outright in section 1. Requirements that no longer apply are removed, not annotated as superseded: git holds the old text, the decision logs hold the why, & a contract carrying its own history becomes unreadable fast. Rejected options join Ruled out.

**A fold leaves nothing open.** Resolve every remaining `[open]` marker in section 1, not just the amendment's — promote, move or cut each — & check any carried-over accessibility block against the general accessibility rules, deleting what they cover.

**Then delete the amendment section**, return the `status` keys to `complete`, & bump the version (§ Versioning). If the change requires consumer action, write the Migration section as you fold the amendment in.

There are no separate amendment CRDs. Git carries the diff, the logs carry the why, the version signals movement, & Migration carries consumer instructions when they are needed. A parallel document would be a fifth copy of facts that already have four homes.

## Keeping it alive

CRDs are updated through Claude Code, alongside the work, in the same commit as the thing that changed. The logs are dated one-liners precisely so that updating is an addition, not a rewrite. If updating a section feels expensive, that is a signal the section is duplicating something with a home elsewhere.

If you ever see an [open] marker in section 1 of a completed CRD with no amendment in flight, someone changed a requirement without closing the thought: treat it as a flag, not furniture.

## Keeping it tight

Every line of a CRD is loaded into an agent's context each time the component is touched, so length is a cost, not a courtesy. The `ds-simplify-prose` skill checks these before every commit.

* **Design decisions & Ruled out entries are one line:** what, & why. A why that needs a paragraph is a requirement, & belongs in section 1.
* **A rejection lives in Ruled out only.** A decision names the choice made & does not re-argue the options it beat.
* **A decision never restates a statement.** Cite the ID & add only the why; if the statement already carries its why, there is no entry.
* **No process.** Build steps, publish state, the date something changed in Figma, draft history & tooling notes belong in the context card, the handover or the commit.
* **No values in statements.** No text style names, px values or token names — Figma holds them. The statement says what the part does, & the component's Figma set is the reference. Section 4's pairing list, where used, is the exception: token names by design.
* **Name doctrine, do not explain it.** "Atom versus consumer", not a paragraph explaining it.
* **The deletion test**, for every line without an ID: delete it — would an agent build the component wrong? If not, it goes.
* **Properties table descriptions stay empty** where Figma says it all.
* **No template leftovers.** An unfilled `* YYYY-MM-DD:` or `* C1:` stub is deleted.

## Quick reference

* One home per fact; link, never duplicate
* Every contract statement: atomic, testable, ID'd; not machine-testable = [review]
* One test (or more) per non-[review] ID; test names cite the ID
* Statements use anatomy nouns & no others
* Rejected options go in Ruled out, with the why
* Provisional = [open] prefix; design closes every one
* Ready for dev is computed: requirements + design_specs complete, design review held, no markers, no open questions; dev review happens during dev
* Conflicts: stop & flag, never silently pick
* Log a decision only if someone could plausibly propose the opposite; supersede & delete, never stack
* Version, once per PR: patch = sections 1–3 change, minor = section complete, 1.0.0 = contract fixed, major = amendment closed (0.x: 1.0.0 only if complete, else minor)
* Changing a shipped component: amendment at the top; closing design folds it in & deletes it, leaving no markers
* Verification output stays in the pipeline, never in the CRD
