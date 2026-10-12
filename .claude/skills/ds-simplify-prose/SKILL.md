---
name: ds-simplify-prose
description: Tighten the prose of a design system document without changing what it says — cut repetition, restated facts, process notes and filler, and shorten what survives. Runs automatically before every commit in ds-crd-write and ds-crd-update, on whatever that session added or changed. Also use when asked to simplify, tighten, trim, shorten or de-waffle a CRD, skill or other design system doc. Not a substance review (ds-crd-review) and not for code.
---

# Simplify prose

A light second pass. The heavy skills carry many rules at once, and prose drifts; this pass
catches it. Every line of a design system document is loaded into an agent's context whenever
the work is touched, so each line has to earn its place.

## Scope

- **Only what the session changed.** When `ds-crd-write` or `ds-crd-update` invokes this skill
  (before every commit, not just the first), check only the lines that session added or
  edited. Diff against the start of the session, or against config § Repo →
  `integration_branch`. Don't rework untouched parts of the document; drift there is a
  separate change.
- **Prose, not substance.** You may shorten, merge, reword and delete lines that repeat or add
  nothing. You may not change what a statement requires, add or remove a requirement, renumber
  IDs, or reverse a decision. If fixing a line would change its meaning, **flag it instead**.
- **Structure stays.** Headings, sections and IDs belong to the owning skill.

## The core: applies to any document

1. **One home per fact.** If the fact lives elsewhere (Figma, code, another CRD, a reference
   doc, the template), link or cite it rather than restating it.
2. **The deletion test.** Delete each line in your head. Would the reader, human or agent, now
   get something wrong? If not, cut it.
3. **Say it once.** When the same point appears in two places, keep it only where it belongs.
4. **No process or history in a reference document.** What happened, when, in which session,
   and what an earlier draft said belong in the commit, the handover or a changelog.
5. **Name doctrine, don't explain it.** Cite a house rule by name; its explanation lives in its
   own home.
6. **No filler.** No preamble, recap, "note that", restated context, or hedging that decides
   nothing.
7. **House style still applies.** Follow config § Writing, and use the terms the CRDs already
   use.

## Profiles

Apply the core, plus the profile for the document in hand.

**CRD**: files in config § Paths → `crd_dir`. Apply `crd_how_to` § Keeping it tight; it is the
home of the CRD rules and wins over anything here. Flag a `C` statement that is advice to the
consumer, but don't move it: moving it to section 6 changes the contract. Cut a section 6 line
that restates a section 1 statement down to the consequence plus the statement's ID
(`crd_how_to` § Section by section, section 6).

**Skill**: apply the core, but **keep the *why* behind a rule.** In a skill, the reason is
often what lets the model apply the rule to a case it doesn't name. Cut repetition, not
reasons.

Other documents (reference docs, tickets): apply the core only.

## Output

- **Edit in place**, then report in a few lines: what you cut or shortened, by section, and a
  before/after word count for the part you touched.
- **List every flag separately**, quoting the line. A flag is a line that breaks a rule but can
  only be fixed by changing its meaning. The person or the owning skill decides.
- If nothing needed changing, say so in one line.
