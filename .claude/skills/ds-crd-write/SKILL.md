---
name: ds-crd-write
description: Write, spec or draft a design system CRD (Component Reference Document) for a component — or fill in a later section of an existing one. Researches the component type, reads related CRDs and the built Figma and code artefacts, then interviews the designer. Use when asked to write, spec, specify, draft, scope, define or document a component; to start or flesh out requirements, specs or specifications for a new or planned component; to capture what a component needs to do; or to complete design specifications, brand validation, CMS behaviour or documentation guidance after design or build. For amending, converting or reconciling an EXISTING CRD use ds-crd-update; for critiquing a draft use ds-crd-review; for reading a CRD to implement against it use ds-crd-read.
---

# Write a CRD

You are helping someone write a **Component Reference Document**. The output is a draft they
own. Your job is to ask the questions that make it complete and to ground every answer in what
already exists — **not to invent requirements for them.**

Assume the person may be new to component design. Explain why a question matters when it isn't
obvious, and never let silence become a decision.

## Before the interview — check the model

If this session isn't running on the most capable model available, say so once and recommend
switching (`/model`). The interview gives UX advice to people who often can't judge it
themselves, so it's only as good as the model. Recommend, never block.

## Read first, every time

| File | Why |
|---|---|
| `.claude/design-system.config.md` | Paths, roles, and which optional CRD sections and features this system uses |
| Config § Paths → `crd_how_to` | The conventions: IDs, markers, versioning, lifecycle, the accessibility boundary, section-by-section guidance |
| Config § Paths → `crd_template` | The skeleton you are filling |
| Config § Paths → `design_foundations` | Figma files, token tiers, mode axes, text styles, spacing |

Don't reconstruct these from memory. If anything contradicts them, they win.

**Never edit the skills or their reference docs from here**: `.claude/skills/`, the config,
`crd-how-to.md`, `crd-template.md`. They belong to config § Roles → `skill_owners` and change in
their own PRs; an edit inside a CRD PR collides with theirs and slips past review. If a rule looks
missing or wrong, tell the person, propose the wording, and ask them to raise it with
`skill_owners`.

**`crd-template.md` defines the shape. Existing CRDs never do.** Use the template's sections and
headings as written. Read other CRDs for *content* precedent (decisions, ruled-out options,
boundaries, naming), never for structure. A CRD that differs from the template is out of date,
not a better example. If you catch yourself following "the shape of X", open the template
instead.

**CRDs live in config § Paths → `crd_dir`, one flat folder.** Filenames follow `crd_filename`
(by default `BasicCard` → `Basic_Card_CRD.md`). Some don't, so match on `name:` front matter too.

**An existing CRD without `format: 2`** is in an older shape. Stop and use `ds-crd-update`;
converting is its job.

## Which job are you doing

**New CRD** — nothing exists. Run every phase.

**Filling a later section** — the CRD exists in the template's shape and design or build has
since happened. Skip to the phase that owns that section; the interview still runs, scoped to it.

| Section | Filled when | Source |
|---|---|---|
| 1 Requirements | Pre-design | The interview |
| 2 Design specifications | Post-design | The component's set in the Figma library |
| 3 Implementation | Pre-dev and during | The developer |
| 4 Brand validation *(if config § CRDs → `brand_validation`)* | Post-design | The component's tokens |
| 5 CMS behaviour *(if config § CRDs → `cms_behaviour`)* | Post-design | The designer |
| 6 Documentation guidance | Post-design and post-dev | Both |

Sections 4 to 6 aren't dev blockers — they serve brand, CMS and docs consumers after the
component ships, so never delete them because a build doesn't need them. Sections 4 and 5 exist
only if the config switches them on; otherwise leave them and their `status` keys out.

## Phase 1 — Context, before asking anything

1. **Read related CRDs.** Glob `crd_dir` for components this will compose with, live inside or
   resemble. Read their Anatomy, Integration, Ruled out and Design decisions in full.
2. **Check nothing already does this job — by purpose, not by name.** Read the Purpose and Scope
   of plausible candidates, and check config § Repo → `component_code_dir`: something may ship
   under a name the CRDs never used.

   If anything is close, name it, link its CRD, and ask whether it (or extending it) would
   serve. If they take it, stop — there's no CRD to write. If not, the reason becomes an
   out-of-scope line naming that component, so nobody has to ask again.
3. **Build a precedent list.** Decisions and Ruled out entries are case law. Collect every one
   that plausibly constrains this component; you'll cite them by name in the interview.
4. **Read the artefacts if they exist.** A component may be built before its CRD is finished.
   - **Figma**: find its set in the library (config § Figma → `component_library`) with
     `search_design_system`. Read variant axes, props, bound tokens and the description.
   - **Code**: its folder under `component_code_dir` — the props interface, style tokens,
     stories.

   Anatomy comes from a real artefact wherever one exists, never from prose.
5. **Note ownership boundaries.** If a parent enforces something on this component, that
   statement belongs in the PARENT's CRD. Work it out now so statements land in the right file.

## Phase 2 — Research, for recognisable patterns

For a known pattern — button, modal, tabs, tooltip, input, toast, accordion, breadcrumb —
research current practice first: how Material, Polaris, Spectrum, Carbon and Atlassian handle
its variants, states and behaviour, and the **WAI-ARIA Authoring Practices** for its interaction
and accessibility.

Research feeds **questions, never requirements**:

- Convergent practice becomes a sharp question: "every major system gives this a focus trap;
  ours needs one unless there's a reason. Is there?"
- Divergent practice becomes a decision for the human, with each side's trade-offs.
- **Precedence: this system's CRD case law beats house convention beats industry practice.**
  Never write an industry default into the contract without the human confirming it applies.

Skip this phase for bespoke components; the closest analogous pattern can still shape questions.

## Phase 3 — Interview

### Keep it short — this is a conversation, not a form

The person is thinking about their component, not about you. Every extra sentence is one they
must read before answering.

- **Ask with the `AskUserQuestion` tool, not as a list in a message.** Anything framed as a
  choice should be: the person picks rather than composes, and can add notes or take "Other". A
  text list of six questions gets one answer back; a picker gets six.
  - Up to four questions per call, two to four options each.
  - **Put the trade-off in each option's description.** That's where someone less familiar with
    components learns why the choice matters.
- **Options in plain text are a short list, not prose.** Two or three, one line each, with the
  trade-off.
  - A house default goes first, marked "(Recommended)".
  - `multiSelect` where several answers can be true — which variant axes apply, which states
    exist.
  - One call at a time. Respond to the answers before asking the next set.
- **Plain text only for genuinely open questions** — purpose, naming the anatomy. Even then,
  offer a proposal to react to: people correct far more readily than they compose. Three to five
  per message, never more.
- **No preamble or echo.** Acknowledge in a few words at most, then ask.
- **One line of context per question, only when it isn't self-evident.**
- **Don't narrate the process** — phases, what you're about to read, that you follow a skill.
- **Take short answers at face value.** Probe only when an answer is ambiguous or contradicts a
  precedent.
- **Don't summarise mid-interview.** Save it for the draft.
- **Report Phase 1 and 2 findings in a few lines**: what constrains this component, nothing else.

Explain *why* a question matters only when the person seems unsure, or when the answer commits
them to something they may not realise — how the component's colour behaves across brands and
themes is the obvious case.

Move on when a topic is settled; circle back rather than block.

1. **Purpose and scope.** What job, what problem, and what this is NOT — every out-of-scope line
   names the component that handles that case instead. Push on the boundary: what looks like
   this but isn't?
2. **Anatomy.** Name the parts together before writing any statement. Every later statement uses
   anatomy nouns and no others; if an answer needs a noun the anatomy lacks, extend the anatomy
   first. Then ask the questions `design_foundations` § System rules raises, for each
   part they apply to.
3. **Variants and options.** Which axes exist and why each earns its place. For every axis ask
   what was considered and rejected — Ruled out material nobody volunteers.
4. **States and behaviour.** Walk the component through its life: resting, hover, focus,
   pressed, disabled, loading, error. Then the edges — longest label, smallest container, slowest
   network. The accessibility boundary (step 6) applies here too: Tab order, reduced motion, RTL
   mirroring and focus indicators are global, whichever section they'd land in.
5. **Content — two destinations.** Copy the component owns or enforces (fixed or translated
   strings, truncation, casing through the text style) becomes ID'd `C` statements, because a
   build test can fail on them. What the consumer should write (voice, length, must and must not)
   is advice, and goes to section 6 under **Content**, un-ID'd. Text case is a house rule (config
   § Writing); don't restate it in either place.
6. **Accessibility — component-specific only.** Read `crd-how-to.md` § What accessibility belongs
   in a CRD first and follow it. Ask about the ARIA attribute and value this component sets, what
   it announces and when, focus order, focus management on state change, and which parts are
   decorative. Don't ask about contrast ratios, keyboard conventions or touch targets; those are
   global (config § Accessibility → `standard` and `rules`).
7. **Integration.** Children, used-in contexts, what this enforces on its children. Apply the
   ownership boundary from Phase 1.
8. **Usage, now — don't skip it.** Draft "when to use / when not to use" from Purpose and Scope
   while it's fresh, add the consumer Content guidance from step 5, and read it back. Each "when
   not to" names its alternative. Write only what the interview produced; never invent patterns.
   No `[review]` markers — section 6 is advice, not contract. Set `docs_guidance: in_progress`;
   the post-design interview finishes it.

### Techniques that matter

- **Cite precedent when it bites.** "The Link CRD ruled out X for this reason. Yours proposes it
  — rename, or overrule?" Overruling is fine; it becomes a logged decision with a why.
- **Borrowed is asked, never assumed.** A statement you'd take from a precedent CRD or from the
  code goes into the interview as a question: "The Link CRD says X. Same here?" Copied in
  silently, it becomes a default nobody chose, and the person can't tell it from what they said.
- **Chase the why.** "Two sizes" is a fact. "Two sizes, no evidenced need for a third yet" is a
  decision worth keeping.
- **A hedge is a prompt, not a marker.** "Probably" or "we might" is often a designer thinking
  aloud. Ask for their view, and push back if a precedent disagrees. Only if they still can't
  decide does it become `[open]`; read the marker back so they know it's recorded as provisional.
- **Separate must from should.** Advice that survives goes to section 6, not the contract.
- **Big uncertainties get the table.** Anything needing an owner and a real decision becomes an
  Open questions row — always ask "by when?". The By column is often the real content.
- **Don't lead.** Present options with trade-offs. Research is context, not a recommendation,
  unless they ask for one.

## Doctrines to hold the draft against

House rules a new author won't know. Apply them as you write, and say so when you do.

- **Atom versus consumer.** An atom owns its visual variants; meaning and positioning belong to
  the components that use it. Name components explicitly — never a bare "CTA".
- **Your system's rules.** Every rule in `design_foundations` § System rules holds; a statement
  that breaks one is a defect, however reasonable it looks.
- **Dark mode follows the theme philosophy** in `design_foundations` § Mode axes.
- **Contrast is validated per theme, not per variant.** In a multi-brand system, brand colours
  cap what's promisable.
  Never guarantee a ratio a brand can't deliver — flag it for per-brand validation (config
  § Accessibility → `contrast_must_pass`).
- **Token mapping tables are the designer's job.** Don't demand one in a CRD.
- **Figma can't express everything.** Where a behaviour is code-only, say so and state what Figma
  approximates it with.
- **WCAG numbers**: 1.4.3 is text contrast, 1.4.11 is non-text, 1.4.4 is Resize Text and not
  contrast.

## Phase 4 — Draft

Write from `crd_template` into `crd_dir`, named per `crd_filename`. Take section headings from
the template verbatim — don't reorder, rename, merge or invent them.

**Front matter** — you own all of it:

- `id`, `name`, `version: 0.1.0`, `format: 2`, `last_updated` today
- `lifecycle: requested`. Moving to `backlog` is a human decision — ask, don't set it
- `tier`, `composition`, `category` — values and rules in config § Paths → `classification`.
  Propose, then confirm with the designer
- `status:` — honest flags. A section you didn't fill is `not_started`
- `figma:` if a set exists in the library; `docs_site:` when the component ships. **`docs_site:`
  is the code link — there is no `code:` key**
- `related_crds` — `dependency` for anatomy children and parents that enforce on this component,
  `precedent` for structurally similar components. Only what an agent must load; loose
  association belongs in Integration prose

**Reciprocal updates, same commit.** Each dependency adds this component to that CRD's "Used
in"; each precedent adds it to that CRD's precedent list. **On the other CRD, update
`last_updated` only — never bump its `version`**: a back-link is front-matter maintenance, not a
requirements change. **Append to its lists, never replace them**, and make no other edit — an
older-format CRD you back-link to isn't converted. Report every file you touched.

**Body**: atomic ID'd statements using anatomy nouns; `[open]` on anything provisional;
`[review]` on anything not machine-testable; every rejection in Ruled out with its why; Open
questions with owner, By and status. Seed Design decisions with decisions already made — dated,
with the why, citing precedent by name — including every `[open]` closed during the interview
(`crd-how-to.md` § Designers: your sections). Log only what someone could plausibly propose the
opposite of; interview churn isn't a decision. Every decision and Ruled out entry follows
`crd-how-to.md` § Keeping it tight.

Leave later sections as empty headings. Empty with an honest status flag beats invented content.

## Phase 5 — Verify, then hand over

Run `ds-simplify-prose` on what you wrote — and again before every later commit, including
fix-up rounds after review — then check:

- [ ] Every statement atomic, testable, and using only anatomy nouns
- [ ] **No values in statements or decisions** (`crd_how_to` § Keeping it tight). Read each one
  for a px or other unit value, a slash-separated token path, or a text style name from
  `design_foundations` § Text styles. Say what the part does instead; Figma holds the value
- [ ] **Every term is one the reader can look up**: an anatomy noun, a term `crd_how_to`
  defines, or one in config § Paths → `glossary`. Any other house term is defined where it
  first appears, or replaced with plain words; tell the person, so they can add it to the
  glossary
- [ ] Nothing came from a precedent or the code without being asked about in the interview
- [ ] No fact appears twice; nothing restates Figma, the code or the reference docs
- [ ] Accessibility passes the boundary test — nothing in it would be true of another component
- [ ] Every unresolved hedge became `[open]`; every non-testable statement carries `[review]`
- [ ] `C` statements are only copy the component owns; consumer copy advice is in section 6
- [ ] Interview step 8 ran: section 6 holds what the interview produced, and nothing it didn't
- [ ] No template stubs left (`* YYYY-MM-DD:`, an empty `* C1:`)
- [ ] Every decision and Ruled out entry has a why
- [ ] Precedent conflicts are honoured or explicitly overruled in Design decisions
- [ ] Every open question has an owner and a By
- [ ] Enforcement statements sit in the right CRD
- [ ] `related_crds` updated in both directions, and you said which files you touched

Present the draft with four lists surfaced, not buried: **the `[open]` markers awaiting design,
the open questions awaiting answers, any precedent overrules, and anything agreed but not yet
written in, with why** (it needs another PR, or it waits on someone). Those are what the human has
to action. An agreed edit that isn't listed reads as done.

Then state the gate plainly: **ready for dev is computed, never stored** — requirements and
design specs complete, the design review held, zero `[open]` markers in section 1, every open
question closed or explicitly deferred. Say what currently stands between this CRD and that.

## Shipping

Every change goes through a branch and a pull request (config § Repo → `direct_push`). If config
§ Tracker → `pr_needs_ticket` is Yes, the PR carries the ticket ref; `ds-ticket` creates the
item. Stage the CRD file explicitly, never `git add -A`.

**The branch and commit sequence, commit messages and the after-PR steps are in
`ds-crd-update` § Ship.** Not restated here; two copies drift apart.
