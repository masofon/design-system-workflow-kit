# Setting up the design system workflow kit

A Claude Code workflow for specifying, designing and handing over design system components: a
**CRD** (Component Reference Document) per component, a Figma build process with clear gates,
and tickets that track each step. It works with any ticketing system and any token
architecture. Your system's details go in one config file, and Claude fills most of it in for
you.

## What you need first

- **Claude Code**, with the official Figma plugin (`figma@claude-plugins-official`).
- **A Figma seat that can edit** your component library.
- **A repo** where the component code and the CRDs will live.
- **Access to your tracker** from Claude Code, through an MCP server or a CLI (`gh` for
  GitHub). Setup walks you through connecting it.
- **At least a basic token architecture in Figma**: published variables that components bind
  to. The kit doesn't build one from scratch yet.

## What's in the kit

```
CLAUDE.kit.md                   ← rules to merge into your repo's CLAUDE.md (setup offers to)
.claude/
  design-system.config.md       ← your system's details
  skills/ds-*/                  ← the workflow, one skill per job
docs/
  crd/crd-template.md           ← the CRD skeleton
  crd/crd-how-to.md             ← how CRDs work: lifecycle, sections, gates
  crd/examples/                 ← worked CRDs for an invented system, to copy the shape from
  design/design-foundations.md  ← your token architecture, plus general Figma know-how
  design/component-classification.md   ← your component categories
  design/accessibility.md       ← global accessibility rules (a WCAG 2.2 AA starter set)
  design/glossary.md            ← your team's terms and the kit's, in plain language
```

| Skill | Job |
|---|---|
| `ds-setup` | Install the kit, connect, fill in the config and foundations, check the setup |
| `ds-ticket` | Raise and move tickets in your tracker (adapters for Azure Boards and GitHub) |
| `ds-crd-write` | Write a new CRD by researching the component and interviewing the designer |
| `ds-crd-update` | Amend, convert or reconcile an existing CRD |
| `ds-crd-review` | Critique a CRD draft, concerns only |
| `ds-crd-read` | Find and apply a CRD when implementing or reviewing code |
| `ds-simplify-prose` | Tighten a doc's prose without changing what it says |
| `ds-design-component` | Build or extend a component in the Figma library |
| `ds-ready-for-dev` | Check a component is ready to hand over, then hand it over |
| `ds-helper` | Answer questions from people using the design system |
| `ds-token-variables` | Edit the token files themselves (design system team only) |
| `ds-design-journey` | Build and lint screens and prototypes, not library components |
| `ds-asset-normalize` | Batch-normalise SVG logos into icon components *(optional)* |

## Setup

Two sentences to Claude: one to install, one to set up.

### 1. Install

Clone or download the kit anywhere **outside** your repo, e.g. `~/Downloads/design-system-workflow-kit`. Open Claude
Code in that folder and say:

*"Install the kit into ~/code/our-repo."*

Claude surveys your repo (its folders, any existing `.claude/` skills, your `CLAUDE.md`, where
your docs live) and proposes where each kit file goes:

- **Skills and the config** go in `.claude/` at your repo root. Claude Code looks for them
  there, so they can't move.
- **The CRD and design docs** go wherever your docs already live, e.g. `docs/design-system/`.
  Claude records the location in the config, so the skills find them.
- **`CLAUDE.kit.md`** gets merged into your `CLAUDE.md` later, not copied as is.
- **README and SETUP** stay in the kit folder; they're for you, not the repo.

You see the full map before anything is copied. Nothing existing gets overwritten: if a skill
name clashes, Claude asks. Everything lands on a new branch.

### 2. Set up

Open Claude Code at your **repo root** (a new session, so the skills load) and say:

*"Set up the workflow kit."*

`ds-setup` works through six phases, reporting after each:

| Phase | What happens | What you do |
|---|---|---|
| 1. Prerequisites | Checks the kit files and the Figma plugin are present | Fix anything missing |
| 2. Connect | Signs in to the repo, Figma and your tracker; works out each address and proves it with a read | Run any sign-in command it hands you; paste your Figma file URLs |
| 3. Config | Works out most values from your repo, tracker and Figma, shows them with evidence, then asks the rest | Confirm or correct; answer the questions |
| 4. Foundations | Reads your token files and library and drafts your design foundations and categories | Confirm, and explain the *intent* behind your structure |
| 5. Finishing | Offers: a custom skill prefix, tracker labels and boards, merging `CLAUDE.kit.md`, one PR for everything | Say yes or no to each |
| 6. Check | Reports anything a skill still can't find | Fix what it flags |

**Nothing it works out is written without your confirmation**, and nothing outward-facing
(labels, boards, PRs) is created without a yes. Stop at any point: saying *"Set up the
workflow kit"* again resumes where you left off.

### Have these answers ready

Claude can work out the technical details. These are people decisions it will ask you for:

- **Who holds each gate:** who publishes the Figma libraries, who awards "ready for dev", who
  approves promoting a component, and who owns the skills.
- **Where questions go:** your design system support channel.
- **Your tags:** which labels you use, and what each one is for.
- **Ticket house rules:** acceptance criteria or not, story points or not, and where new items
  go.
- **Contrast:** which brand and theme combinations must pass, and which have latitude.
- **Access:** who can read the repo and the docs site.
- **Which optional parts you use:** stable components shipped ahead of a design, and amendments
  to shipped components.
- **Your page layout:** keep your existing component-page layout, or use the kit's.
- **On GitHub:** whether you use milestones, a Projects board or labels for status, and whether
  PRs use closing keywords. Setup recommends an answer for each.

### Later

- *"Check the setup"* re-runs phase 6 on its own, read-only.
- Don't recognise your tracker? Setup offers to draft an adapter from
  `.claude/skills/ds-ticket/adapters/README.md`.

## Making it yours

Everything in the kit is editable. The config holds your facts, but the skills also carry house
rules: opinions about how a design system should be run, such as one CRD per component, sentence
case, and placing components by what they do rather than the feature they came from. Keep them,
change them, or drop them. A changed rule belongs in the skill that owns it, so every session
picks it up.

## Using it

Most work starts with one sentence to Claude:

- *"Spin up tickets for a Tooltip component."*
- *"Write the CRD for Tooltip."*
- *"Design the Tooltip component in Figma."*
- *"Is Tooltip ready for dev?"*

Each skill tells you when it hands over to the next one.
