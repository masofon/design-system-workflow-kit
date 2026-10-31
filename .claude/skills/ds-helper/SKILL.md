---
name: ds-helper
description: Answer questions from people USING the design system — feature-team designers, devs and product people — by reading the CRDs, the published Figma libraries and the code, then replying with a short answer, the component's lifecycle and links to the CRD, Figma and the docs site. Use when someone asks which component to use for something, which token to use, whether a component supports dark mode or another theme or brand, whether something is built yet or when it is coming, what props a component takes, how it may be used, which icon to use, what accessibility a component handles and what is left to them, why something does not exist, or how to set up access to the sources. Read-only. Not for writing or editing a CRD (ds-crd-write, ds-crd-update), building in Figma (ds-design-component), or implementing or reviewing code against a CRD (ds-crd-read).
---

# Design system helper

Answers questions from people consuming the design system. **Read-only**: it changes no CRD,
Figma file, ticket or file.

**This skill owns routing, not rules**: which source answers which question, and the shape of
the reply. Answer from the sources, not from a summary here.

## 0. Before answering

1. **Freshen the clone.** If there is a local clone of config § Repo → `slug`, pull config § Repo
   → `integration_branch` fast-forward only. Answering from a stale clone is the commonest wrong
   answer. No clone? See § Setting up access.
2. **Name the component.** Map the question to one or more CRDs (§ Sources). If it names no
   component ("what do I use for an on/off setting?"), shortlist candidates by `name`, Purpose
   and Scope, then read them in full.
3. **Ask only if the answer changes.** One clarifying question at most, and only when the
   candidates genuinely diverge ("in a form you submit, or a setting that applies at once?"),
   asked per `.claude/CLAUDE.kit.md` § Asking questions.

## Sources, and which one wins

Read in this order. Each row lists where to look, best first; use the first that is available.
A row whose config value is `—` doesn't exist in this system: skip it.

| Source | Where | Answers |
|---|---|---|
| **CRD** | Clone: config § Paths → `crd_dir` · else the repo host, same path on `integration_branch` | Purpose, scope, lifecycle, usage, accessibility boundary, what was ruled out, open amendments |
| **Code** | Clone: config § Repo → `component_code_dir` · else config § Sources → `package_components` in `node_modules` | Props and API of a **built** component; whether a component exists in code at all |
| **Code Connect** | Clone: config § Sources → `code_connect_files` | The library node URL for a built component, and the Figma-prop → code-prop mapping |
| **Token CSS** | Clone: config § Sources → `token_css` · else the same file inside `package_tokens` | Whether a code token exists, and its exact name |
| **Icons** | Clone: config § Sources → `icons_dir` · else `package_icons` · else the icon components in the Figma library | Which icons exist, by name |
| **Figma** | Figma MCP, published libraries only (`design_foundations` § Files) | A component's node link; confirming mode support; anything the CRD leaves blank |
| **Docs site** | config § Repo → `docs_site`; readable by an agent only if config § Sources → `docs_site_access` says so | Otherwise a link only — § Links |

**Who wins.** The CRD is the authority for intent, lifecycle and usage. For **code questions on a
built component** — props, defaults, token names — the code wins. Figma is read to get a link, to
confirm mode support, or when the CRD is blank or `active` on the point.

**Never silently pick a side.** If two sources disagree, give both, say which one you'd trust and
why, and flag it as drift for the design system team (§ Gaps).

**Figma sees only what is published** (`design_foundations` § Publish gates). An unpublished
change is invisible to you. When Figma lags the CRD, say "not published yet", not "doesn't
exist".

### Finding the CRD

Glob `crd_dir` and match the front matter `name` loosely (case, spaces, underscores), not the
filename. The filename pattern is config § Paths → `crd_filename`. If config § CRDs → `amendments` is Yes, check for an
amendment in flight: a `# Amendment` section above section 1.

## By question type

Read the named place and answer from it. Section numbers follow config § Paths →
`crd_template`.

- **Which component for X?** Purpose, Scope and `## Ruled out` across the candidates; §4 Usage
  for when to pick one over another. Name the one to use and the one-line reason; mention the
  runner-up only when the choice is close.
- **Which token for Y?** Read `design_foundations` § Component tokens and § Finding tokens
  first, every time. Then look up the real name: `search_design_system` against the published
  semantic library for designers, the token CSS for devs. Never a placeholder name, never a hex
  value. State whichever rule from those sections applies, one line each, because these are what
  people get wrong: typically container versus page background tokens, and that a component
  token beats a semantic one.
- **Does Z support dark mode, or another brand?** Modes are `design_foundations` § Mode axes;
  say which axis you are answering. The CRD's §1 says what the component supports. Confirm in
  Figma only if the CRD is silent or `active`.
- **Is W built? When is it coming?** `lifecycle` (meanings in `crd_how_to` § Lifecycle) plus any
  open amendment. You have no dates: don't guess one. Point to the support channel for timing.
- **What props does it take?** Built: the component's types in code, plus the Code Connect
  mapping. Not built: §2 Properties in Figma, labelled "planned — the dev team owns the final
  API".
- **How may I use it?** §4 Usage and Content, §1 Functional requirements, `## Gotchas`.
- **Which icon?** Search the icon names; give the exact name and the import or Figma component.
  If none fits, say so. Never suggest a typed glyph.
- **What accessibility does it handle?** The CRD's Accessibility section says what the component
  owns. Spell out what is left to the consumer, such as labels, live regions and focus return.
  The standard is config § Accessibility → `standard`.
- **Why don't we have X?** `## Ruled out` in related CRDs, and any CRD with `lifecycle:
  rejected`. Quote the recorded reason. Don't invent one.

## Answer shape

Answer first, in one to three sentences. Then one status line, then links. Go into detail only
when asked.

**Short is the point.** No preamble, no restated question, no explaining how the design system
works unless that is the question. A rule the asker must follow gets one line. Offer anything
they didn't ask for in one line at the end ("Want the hover and pressed tokens too?").

```
**Use Switch** — for an on/off setting that applies at once. For a choice submitted with a
form, use a checkbox.

Status: requested · not designed or built yet

- CRD: <repo link>
- Figma: — (not designed)
- Docs: — (not built)
```

- **Status line:** lifecycle in plain words, plus the CRD version for anything `active` or
  `built`. Add what's relevant to the question, such as "light theme only" or "no dark mode
  yet".
- **Write for the asker.** A product person gets no token names unless they ask. A dev gets exact
  names. Keep the CRD's terms, but gloss any jargon once.
- **Open amendment:** when the question is about building on the component, open with a
  one-line "this is changing", saying what.
- **Not built:** name the nearest built alternative (`related_crds.precedent`, or the closest
  built component by Purpose). If config § CRDs → `stable_components` is Yes and a `stable:`
  block exists, give its `location`, since that is usable today.
- **Uncertain:** say which source you couldn't read or which section was blank. Never fill the
  gap from general design-system knowledge and present it as this system's answer.

## Links

- **CRD:** the file's URL on config § Repo → `url`, on `integration_branch`.
- **Figma:** best first — the Code Connect URL, then the CRD `figma:` field, then a node found
  through the Figma MCP. If none exists, write "not designed".
- **Docs site:** if the CRD `docs_site:` field holds a full URL, use it. Otherwise build it from
  config § Sources → `docs_site_link_pattern`, if set. For Storybook, the id is the stories
  `title` lowercased, with every run of spaces or `/` turned into one hyphen; it does **not**
  split camelCase (`Feedback/Badge` → `feedback-badge`, `Data Display/AvatarGroup` →
  `data-display-avatargroup`). Append `--docs` only if the meta has `tags: ['autodocs']`;
  without it, give the sidebar path (Overlays › Modal). No stories file means no link: write
  "not built". **Tag any link you built and couldn't open "(built from the story title, not
  opened)".**

## Gaps and escalation

When the sources don't answer, or they disagree:

1. Say so plainly, and say what you did find.
2. Send the asker to config § System → `support_channel`.
3. Offer a ready-to-paste post: the question, what the sources said, and the gap ("Switch CRD
   doesn't say whether it supports dark mode"). That's how the docs improve from real questions.

Write nothing yourself: no ticket, no CRD edit, no comment.

## Setting up access

When a source can't be read, or the person asks how to get set up, tell them what each one needs.

- **The repo (CRDs, code, Code Connect).** Who can read it and how is config § Sources →
  `repo_access`. With access, either clone it or sign in to the repo host's CLI (`gh auth login`
  on GitHub) and read files through it. Without access, ask in the support channel.
- **The packages.** config § Sources → `package_components`, `package_tokens`, `package_icons`.
  A consumer app that already has them in `node_modules` covers tokens, icons and built code with
  no extra access. CRDs are not in them.
- **Figma.** The Figma MCP comes from the `figma` Claude Code plugin. It authenticates as the
  person, so they need a Figma seat with access to the libraries (config § Figma → `org`).
- **Docs site.** config § Sources → `docs_site_access`. If Claude can't open it, links are for
  the person to follow.
