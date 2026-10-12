---
name: ds-setup
description: Install the design system workflow kit into a team's repo and set it up, or check an existing setup — place the kit's files into the repo's existing structure, connect and sign in to the repo, Figma and the tracker, fill in the design system config and the design foundations by deducing what the sources show and asking only for the rest, run the tracker adapter's setup choices, then verify every skill can find what it needs. Use when installing the kit from its download or template folder into a repo, when setting up the kit, connecting it to a repo, Figma or a tracker, filling in or updating the design system config, when asked to "check the setup", or when a skill reports a missing config value. Re-running picks up where the last run stopped.
---

# Set up the workflow kit

Gets a team from "we've downloaded the kit" to "every skill can run". `SETUP.md` is the human
version of this flow.

**Three modes:**
- **Install** ("install the kit into {repo}"): phase 0, run from the **kit's own folder**. It
  places the kit into the team's repo, then hands over to setup.
- **Setup**: phases 1–6, run from the **team's repo root**. Re-running resumes: completed
  connections are re-checked, not redone, and only empty config values are revisited.
- **Check** ("check the setup"): phase 6 only, read-only.

**Which mode?** A folder holding `SETUP.md` and `CLAUDE.kit.md` is the kit's download folder:
run install. A repo with the kit's skills in `.claude/skills/` and no `SETUP.md`: run setup.

## Rules for the whole run

- **Work out, confirm, then write.** Show each worked-out value with its evidence before
  writing it. Batch confirmations, one section at a time.
- **Ask with AskUserQuestion**, several questions at once, and recommend an option where the
  evidence leans one way.
- **Contradictions are questions.** If the sources disagree, show both and ask. Never pick.
- **Outward-facing writes need an explicit yes**, every time: creating labels, a Project, a
  GitHub Action, a branch, a PR. Reading never does.
- **Sign-in that needs a browser or a prompt is the person's.** Hand it over as
  `! <command>`, in short commands rather than one long line, then check it worked.
- **One phase at a time.** Finish it, report in a few lines, carry on.
- **Repo writes** follow the config once it's filled: a branch and a PR into
  `integration_branch` if `direct_push` is No.

## Phase 0: Install

Run from the kit's folder. Never copy the folder wholesale: each file goes where *that repo*
keeps things.

1. **Find the target repo.** Ask for its path (or URL, to clone). Confirm it's a git repo and
   that the person can push a branch to it.
2. **Survey it** (read-only):
   - Repo layout: monorepo or single package, where the docs live, and any existing
     `docs/`, `documentation/` or wiki folder.
   - `.claude/`: existing skills (any name clashes with `ds-*`?), `settings.json`, agents.
   - `CLAUDE.md` or `AGENTS.md` at the root.
   - Existing component specs or CRDs, in any format. Note them; they're converted on touch
     later, never in bulk.
3. **Propose a placement map** and show it as a table, *kit file → repo path*, marking each one
   **new**, **merge** or **clash**:

   | Kit file | Goes to | Can it move? |
   |---|---|---|
   | `.claude/skills/ds-*/` | `.claude/skills/` at the repo root | No. Claude Code looks for skills there |
   | `.claude/design-system.config.md` | `.claude/` at the repo root | No. Every skill looks for it there |
   | `docs/crd/` (template, how-to, README, examples) | Beside the team's other docs, e.g. `docs/design-system/crd/` | Yes. Config § Paths records where |
   | `docs/design/` (foundations, classification, accessibility) | Beside the CRDs, e.g. `docs/design-system/` | Yes. Config § Paths and § Accessibility → `rules` record where |
   | `CLAUDE.kit.md` | Merged into the root `CLAUDE.md` in phase 5, never copied as is | |
   | `README.md`, `SETUP.md` | Not installed: they're for the person installing | |

   - **A skill name clash**: never overwrite. Offer the kit's skill under a new prefix (config
     § System → `skill_prefix`), or a side-by-side diff for the person to decide.
   - **A monorepo**: `.claude/` still goes at the repo root, where Claude Code is started.
     Ask where design system docs belong if there's more than one candidate.
4. **Get a yes on the map**, then create a branch in the target repo, copy the files, and
   write the chosen doc locations into the target's config § Paths and § Accessibility → `rules`. Don't commit yet; phase 5
   commits everything together.
5. **Hand over.** Skills load from the folder Claude Code starts in, so setup continues in a
   new session at the target repo root. Give the person two short commands (`cd` to the repo,
   start `claude`) and the sentence to say: *"Set up the workflow kit."*

## Phase 1: Prerequisites

Confirm, from the repo root:
- The kit is installed: `.claude/design-system.config.md`, `.claude/skills/ds-*`, and the CRD
  and design docs at the paths in config § Paths. If anything is missing, point back to phase 0,
  run from the kit's folder.
- Claude Code has the official Figma plugin. If not, stop and say how to add it.
- The model. Recommend a top-tier model for setup; much of it is judgement.

## Phase 2: Connect

Stop at the first connection that fails, fix it, then carry on.

| Connection | Sign-in check | Address |
|---|---|---|
| **Repo** | The repo host's CLI auth status (`gh auth status` on GitHub) | **Work it out** from `git remote get-url origin`, then confirm. No remote, or several: ask |
| **Figma** | `whoami`: a seat that can edit in the org that owns the library | **Ask** for the URL of the component library and of each token file. Take the keys from the URLs |
| **Tracker** | The adapter's § Setup | Work it out where the adapter allows, otherwise ask |

**Choosing the tracker:** look for ticket refs in recent commit messages and PR bodies
(`AB#123` is Azure Boards, `ABC-123` is Jira or Linear, `#123` is GitHub). Propose what you find,
and confirm. Then read the matching adapter in `.claude/skills/ds-ticket/adapters/`. If none
exists, offer to draft one from `adapters/README.md`, and mark tracker setup unfinished until
it's proven.

**Prove every address with a read** before you keep it: list a few issues, open each Figma
file, list the repo's recent PRs.

## Phase 3: Fill in the config

Go section by section through `.claude/design-system.config.md`, in file order. For each one:

1. **Work out** what the sources show (table below), noting the evidence.
2. **Keep the defaults** already in the file unless evidence contradicts them.
3. **Confirm** the drafted values in one batch, then **ask** what's left.
4. **Write** the section, then move on.

| Section | Work out from |
|---|---|
| **System** | `name` and `org` from the repo name and the Figma file names. Always ask for `support_channel` |
| **Roles** | Suggest candidates from who publishes the Figma libraries (version history) and who merges PRs. **Always ask**: gates are people decisions |
| **Repo** | `host`, `slug`, `url` from the remote. `default_branch` from the repo settings. `integration_branch` from the base branch of recent merged PRs. `direct_push` and any required ticket check from branch protection and rulesets. `component_code_dir`, `token_export_dir` and `docs_site` by scanning the tree, the README, the repo homepage and any Storybook config. `commit_format` from commitlint, husky or similar config, else recent commit messages |
| **Paths** | Where phase 0 put the kit's docs |
| **Sources** | Package names from `package.json` files. `token_css` from the token build output. `icons_dir` from the tree. `code_connect_files` from any `*.figma.*` files. `docs_site_link_pattern` from the Storybook config and one live URL. Ask about `repo_access` and `docs_site_access` |
| **Tracker** | Run the adapter's setup choices first (e.g. GitHub § 0); they fill *Expressed as*. Roles and stages from the tracker's own types, states, issue types or Project status options. `ticket_ref` from history. **Tags**: list the labels already in use, with counts, and ask which form the taxonomy and what each is for. Always ask the house rules |
| **CRDs** | Existing CRDs, if any. Ask whether the team uses stable components, brand validation and CMS behaviour. **An existing spec or CRD corpus** in another format is converted on touch by `ds-crd-update`, never in bulk: note where it lives and move on |
| **Figma** | `org` from `whoami`. Library conventions from the library's existing pages and component sets: set naming, page naming, WIP and protected pages, canvas colour, any status symbols, the reference page (the most recent one carrying the ready-for-dev symbol). `assets_page` and `app_icon_name` from existing icon components. **Ask whether the library has documentation utility components** (section header, doc row, anatomy pin) for `doc_utilities`; if not, the design skills build them once, locally. Ask for whatever else the library doesn't show yet. Then run **§ Page layout** below |
| **Tokens** | Deprecation naming and any no-export marker from existing variable names |
| **Accessibility** | List the brand × theme modes from the token files, and ask which must pass contrast and which have latitude. Look for an existing accessibility rules file or agent in the repo. If there is one, ask: point `rules` at theirs, merge the kit's starter into it, or keep both. If there isn't, keep the kit's starter (`docs/design/accessibility.md`) and say it's theirs to edit |
| **Writing** | Text case from existing text content and text styles; otherwise keep the defaults |

### Page layout

Run inside the Figma row, before writing config § Figma → Library conventions.

1. **Sample** several existing component pages: the most complete, or the most recent.
2. **Detect a consistent layout**: the top-level frames and their order, frame naming and
   headings, fills, spacing and radius, any context card, any doc utility components in use.
3. **Show what you found**, with a screenshot of one representative page if you can.
4. **Ask** (AskUserQuestion), recommending from the evidence:
   - **(a) Keep my layout.** Record their conventions into the Library conventions keys, set
     `page_layout: team` and `reference_page` to the page that shows it best. Their frames
     needn't match the kit's four; `doc_frames` lists theirs. Recommend this when the pages are
     consistent.
   - **(b) Use the kit's default.** Set `page_layout: kit`; the kit's defaults in Library
     conventions stand.
5. **Inconsistent pages, or none:** say so and recommend (b), offering to record (a) from one page
   the person nominates.

## Phase 4: Fill in the design foundations

`config § Paths → design_foundations`, Part 1 (*Your system*). Every design skill reads it
before touching Figma, so it can't stay empty.

Read the token files and the library with read-only `use_figma` and `search_design_system`
calls: collections, modes, tier aliasing, component token naming, text style groups, spacing
variables, effect styles, the icon wrapper frame and where icon colour binds. Draft each Part 1
heading from what's there, confirm, and write. **Leave Part 2 alone**: it's generic.

**Ask about intent, not structure.** Why a mode exists and which tier components may bind are
design decisions the files can't show.

**Then interview for § System rules.** Single brand or several, and what may vary between them;
colour families that travel together; customer theming; platforms; anything else that changes
how a component is specified. Propose candidates from what the token files show (brand modes,
status families), then turn each answer into a rule and the question it raises.

Then `config § Paths → classification`: propose categories from the library's page or section
headers and confirm.

**No token architecture yet?** The kit assumes at least published variables that components
bind; say so. Record what exists, mark the gaps `[open]` in foundations, and carry on. The design
skills stop at those gaps rather than guess.

## Phase 5: Finishing touches

Offer each one; do each only on a yes.

- **Skill prefix.** If config § System → `skill_prefix` isn't `ds`, rename every skill folder,
  each `name:` and every cross-reference, then grep to prove no old name survives.
- **Tracker scaffolding.** Whatever the adapter's choices call for: labels for every tag (with
  descriptions from the config), a Project's status options, an optional ticket-ref check.
  Outward-facing: list exactly what you'll create, and get a yes.
- **CLAUDE.md.** Offer to merge `CLAUDE.kit.md` into the repo's `CLAUDE.md` (or create one).
  Show the result before writing.
- **Example CRDs.** Keep them as reference until the team has written its own, then offer to
  remove them.
- **Commit.** Everything setup wrote goes in one branch and PR, with a ticket ref if
  `pr_needs_ticket` is Yes (raise the ticket with `ds-ticket`).

## Phase 6: Check

Read-only. Report **block** findings first, then **warn**, then what passed in one line.

**Block:**
- An empty config value that a skill cites. Grep the skills for `config §` citations and
  check each key has a value or `—`.
- A path in config § Paths that doesn't resolve.
- No adapter for the configured tracker, or the tracker unreachable.
- A Figma file in the config or foundations that won't open, or no seat that can edit.
- An empty Part 1 heading in foundations.

**Warn:**
- A `config §` citation in a skill whose key doesn't exist in the config. This is a kit bug:
  name the skill and the key.
- Adapter sections still marked *(verify)*.
- Config values left at a kit default that the sources contradict.
- Example CRDs still present after the team has written its own.

**Not applicable** (one line, not a finding): features the config switches off, e.g. stable
components or brand validation.

End with what to do next: usually *"Spin up tickets for {a component}"* or *"Write the CRD for
{a component}"*.
