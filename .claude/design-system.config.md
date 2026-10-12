# Design system config

The one place system-specific facts live. Skills hold the **process** and cite this file by
section and key, e.g. *config § Tracker stages → `verify`*, instead of naming a board state, a
Figma file or a person.

Written for people and machines alike: plain tables, one fact per row. **Fill in the Value
column; leave the Key column alone**, because the skills look values up by key.

- **Empty** means not filled in yet. A skill that needs it stops and asks, and `ds-setup`
  reports it.
- **`—`** means it doesn't apply to this system.

`ds-setup` fills most of this from your repo, tracker and Figma files, then asks for the rest.

## System

| Key | Value | Notes |
|---|---|---|
| `name` | | The design system's name, used in prose: "the {name} library" |
| `org` | | The organisation that owns it |
| `skill_prefix` | ds | Skills are named `{prefix}-crd-write` etc. |
| `support_channel` | | Where questions the sources can't answer go: a Slack or Teams channel, with link |

## Roles

Who holds each gate. Skills pause and ask the role; they never act for it. Name a person or
group, never an email.

| Key | Value | Gate |
|---|---|---|
| `publisher` | | Publishes the Figma libraries: every publish gate |
| `ready_for_dev_awarder` | | Sole owner of the ready-for-dev mark |
| `promotion_gate` | | Promotes a stable component into the core library |
| `skill_owners` | | Contributors propose edits to skills and reference docs; these people make them |

## Repo

| Key | Value | Notes |
|---|---|---|
| `host` | | GitHub, GitLab, Azure Repos, Bitbucket |
| `slug` | | e.g. `acme/design-system` |
| `url` | | |
| `default_branch` | | e.g. `main` |
| `integration_branch` | | The branch PRs merge into. Same as `default_branch` if there's no separate one |
| `direct_push` | | Yes / No. No = every change, docs included, goes via branch and PR |
| `pull_at_session_start` | Yes | |
| `component_code_dir` | | Where built components live |
| `token_export_dir` | | Tokens generated for code, if any. Never the authority on what exists in Figma |
| `docs_site` | | Storybook or equivalent |
| `commit_format` | | The repo's commit message convention and any hooks enforcing it, e.g. Conventional Commits via commitlint |

## Paths

Relative to the repo root.

| Key | Value | Notes |
|---|---|---|
| `crd_dir` | `docs/crd/` | One flat folder. State lives in front matter, never in the folder |
| `crd_template` | `docs/crd/crd-template.md` | |
| `crd_how_to` | `docs/crd/crd-how-to.md` | |
| `crd_filename` | `{Name}_CRD.md` | |
| `classification` | `docs/design/component-classification.md` | Defines the CRD's `tier`, `composition`, `category` values |
| `design_foundations` | `docs/design/design-foundations.md` | Your token architecture, text styles, spacing and Figma facts |
| `skills_dir` | `.claude/skills/` | |

## Sources

Where people and agents find the built system. `ds-helper` reads these to answer questions.
`—` for anything you don't have.

| Key | Value | Notes |
|---|---|---|
| `package_components` | | Published package name for components, e.g. `@acme/components` |
| `package_tokens` | | Published package name for tokens |
| `package_icons` | | Published package name for icons |
| `token_css` | | Path to the generated token CSS |
| `icons_dir` | | Icon source folder |
| `code_connect_files` | | Glob for Code Connect files, if used |
| `repo_access` | | Who can read the repo, and how |
| `docs_site_access` | | Whether an agent can read the docs site, and any sign-in it needs |
| `docs_site_link_pattern` | | How to build a link to one component's docs, e.g. Storybook's `?path=/docs/{category}-{name}--docs` |

## Tracker

Any ticketing system works. The skills speak in the workflow's own terms (*container*, *work
item*, *stage*, *tag*), and this section says how your tracker expresses each. Tracker mechanics
(field names, API quirks) live in the tracker's adapter beside `ds-ticket`, not here.

| Key | Value | Notes |
|---|---|---|
| `tracker` | | Azure Boards, GitHub Issues, Jira, Linear, Trello… |
| `adapter` | | e.g. `ds-ticket/adapters/github.md` |
| `access` | | MCP server name, or CLI. Never a personal token in chat |
| `auth_fix` | | What to tell someone when auth fails, e.g. `! gh auth login` |
| `board` | | Whatever addresses one board: repo, project, team, area path, Project number |
| `new_items_go_to` | | e.g. Backlog. Never straight into a sprint |
| `item_url` | | URL pattern with `{id}` |
| `ticket_ref` | | How a commit or PR cites an item: `#{id}`, `AB#{id}`, `{KEY}-{id}` |
| `pr_needs_ticket` | | Yes / No, and the CI check that enforces it, if any |
| `ticket_ref_in_commit` | | Yes / No |
| `people_domain` | | Tracker identities, if different from email |

### Tracker item roles

| Key | Value | Expressed as | Used for |
|---|---|---|---|
| `container` | | Type or label; how work items attach (parent link, sub-issues) | One per component, or per amendment |
| `work_item` | | | One per PR |
| `defect` | | | |
| `subtask` | | | Optional |

### Tracker stages

The workflow's fixed stages, and how your tracker expresses each: a state, a column, a label or
a project field. Skills move items by stage.

| Key | Value | Expressed as | Meaning |
|---|---|---|---|
| `new` | | | Raised, not started |
| `ready_for_dev` | | | Handed to dev: the container and the develop item |
| `in_progress` | | | Being worked, or PR in draft |
| `in_review` | | | PR open and ready for review |
| `verify` | | | PR merged: ready to verify, not done |
| `done` | | | |
| `blocked` | | | |
| `removed` | | | The tracker's nearest thing to delete |

| Rule | Value |
|---|---|
| Only move forward | Yes |
| Never move out of | `blocked`, `done`, `removed` |
| Never change sprint or iteration | Yes. Sprint placement is the human's |

### Tracker spin-up

The items raised when work starts. `{Name}`, `{n}` and `{summary}` are filled in. Change the
wording freely; keep the order.

| Key | Container | Work items, in order |
|---|---|---|
| `component` | `Component: {Name}` | Write the {Name} CRD · Design the {Name} component · Develop the {Name} component |
| `amendment` | `Component: {Name} — Amendment {n}: {summary}` | Write Amendment {n} in the {Name} CRD *(only if not written)* · Design {Name} Amendment {n}: {summary} · Fold Amendment {n} into the {Name} CRD · Develop {Name} Amendment {n}: {summary} |

### Tracker tags

Tags, labels: whatever your tracker calls them. Skills suggest from this list, confirm with the
person, then apply. They never auto-apply and never invent one.

| Tag | For |
|---|---|
| | |

| Rule | Value |
|---|---|
| Never use | Tags true of every item on the board, e.g. the system's own name. They filter nothing |
| Casing | Match the existing tag exactly |

### Tracker house rules

| Rule | Value |
|---|---|
| Acceptance criteria checklists | |
| Assign to | Whoever did the work. Never default to the person running the session |
| Only modify your own items | Yes. Check assignee and creator before any write |
| Show before creating | Yes. A create is visible to the whole team at once |
| Story points | |
| Closing line on component items | — |

## CRDs

| Key | Value | Notes |
|---|---|---|
| `meaning` | Component Reference Document | |
| `format` | 2 | |
| `lifecycle` | requested · backlog · active · stable · built · deprecated · rejected | Defined in `crd_how_to` § Lifecycle |
| `stable_components` | | Yes / No. Implementations shipped ahead of a designed core component |
| `amendments` | Yes | Changes to a shipped component go through an amendment block |
| `brand_validation` | | Yes / No. Section 4: runtime contrast checks when customers or sub-brands supply their own colours |
| `cms_behaviour` | | Yes / No. Section 5: how a content editor configures the component in a CMS |
| `simplify_before_commit` | Yes | `ds-simplify-prose` runs on every CRD commit |

## Figma

| Key | Value | Notes |
|---|---|---|
| `org` | | A seat that can edit here is needed to write |
| `component_library` | | File name and key |
| `token_files` | See `design_foundations` § Files | Tiers, keys and modes live with the design facts |
| `consumers_see_published_only` | Yes | Every token or library change needs `publisher` before anything downstream sees it |

### Library conventions

| Key | Value | Notes |
|---|---|---|
| `page_layout` | | `kit` \| `team`. `team` = the conventions below describe the team's own layout, recorded at setup; `reference_page` is the source of truth |
| `reference_page` | | The page whose layout every new component page copies. Under `kit`, the most recent page carrying the ready-for-dev symbol |
| `set_name` | `{category}/{name}` | Lowercase kebab-case |
| `stable_set_name` | `{category}/stable/{name}` | |
| `wip_set_name` | `{category}/wip/.{name}` | A leading dot keeps it out of library publishing |
| `page_name` | `❖  {Name} - {symbols}` | |
| `wip_pages` | Backlog, Rough Components | Pages where unfinished work lives |
| `protected_pages` | | Pages skills never touch, e.g. foundations or utilities |
| `canvas_background` | | Page background colour, if standardised |
| `context_card` | | A frame at the top of each component page carrying its metadata |
| `doc_frames` | Component · Anatomy · Usage · Modes | Top-level frames on a component page, left to right |
| `frame_names` | `{Component}`, `{Component} — anatomy`, `— usage`, `— modes` | |
| `frame_heading` | `0N · {Name}` | e.g. `01 · Button`, `02 · Anatomy` |
| `frame_spacing` | 160 | Horizontal gap between top-level frames, px |
| `frame_fill` | | Token for the doc frames' background. Usually the page background token |
| `frame_radius` | 16 | px, or a radius token |
| `frame_spacing_tokens` | | Gap and padding tokens used inside doc frames |
| `specimen_tile_outset` | 40 | px a backdrop tile extends beyond each variant |
| `specimen_tile_gap` | 48 | Minimum px between backdrop tiles |
| `responsive_widths` | 560 · 440 · 320 | Specimen widths for desktop · tablet · mobile in the modes frame |
| `status_symbols_page` | | A page in the library that explains the symbols below |
| `doc_utilities` | | Components the documentation frames are built from (section header, doc row, anatomy pin…), or `—` to build them locally |
| `assets_page` | | The page holding asset artboards (logos, app icons) |
| `app_icon_name` | | Naming pattern for app or logo icons, e.g. `icons/app-{name}` |
| `section_header_component` | | Component used for artboard headers on asset pages |

### Page status symbols

Stack after the dash in the page name.

| Key | Symbol | Set by |
|---|---|---|
| `ready_for_dev` | ✅ | `ready_for_dev_awarder` only |
| `amendment_in_design` | 🔄 | Skill. Removed when the amendment ships |
| `in_progress` | 🖍️ | Skill |
| `waiting` | ⏳ | |
| `deprecated` | ⚠️ | |
| `new` | 🆕 | |

## Tokens

The architecture itself (tiers, collections, modes, naming, spacing scales) is design knowledge
and lives in `design_foundations`. This section holds only the switches skills need.

| Key | Value | Notes |
|---|---|---|
| `deprecated_name` | `{group}/do-not-use-{leaf}-deprecated` | |
| `no_export_marker` | | A prefix or tag that keeps a mode or variable out of the code export, if any |

## Accessibility

| Key | Value | Notes |
|---|---|---|
| `standard` | WCAG 2.2 AA | |
| `rules` | `docs/design/accessibility.md` | The global rules every component meets. The kit ships a WCAG 2.2 AA starter set; replace or extend it, or point at your own |
| `contrast_must_pass` | | Every brand × theme mode combination that must pass |
| `contrast_latitude` | — | Demo or internal brands: make passable, don't block |

## Writing

| Key | Value |
|---|---|
| `text_case` | Sentence case for all text content. Casing is presentation, applied by the text style |
| `case_exceptions` | Acronyms, hex values, single-glyph specimens |
| `gloss_ticket_ids` | Yes. Every mention reads `#123 (what it is)` |
| `report_style` | Concerns first. Always say what was deliberately not changed, and why |
