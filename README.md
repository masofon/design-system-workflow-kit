# Design system workflow kit

A Claude Code workflow for running a design system's component pipeline, from request to
hand-off:

- **A CRD per component.** One Component Reference Document holds what the component must do,
  how it's designed and how it's built. People and agents both read it.
- **Figma with gates.** A repeatable build process for the component library, with checks
  before anything is called ready.
- **Tickets that follow the work**, in whatever tracker you use.

Your system's specifics live in one config file. The skills hold the process.

> **Early release.** The workflow has run for months on a real design system; this generic
> version is new. Setup hasn't been run on many repos yet, and the GitHub tracker adapter is
> unproven (its uncertain steps are marked *(verify)*). Expect rough edges, and please report
> them as issues on this repo.

## What you need

- **Claude Code**, with the official Figma plugin.
- **A Figma seat that can edit** your component library.
- **A design system built on Figma variables**: published tokens that components bind to. The
  kit can't build a token architecture from scratch yet.
- **A repo** for the CRDs, usually beside your component code.
- **A tracker** Claude Code can reach: GitHub Issues (via `gh`) or Azure Boards have adapters;
  for anything else, Claude drafts one with you.

Full details in [`SETUP.md`](SETUP.md).

## Getting started

1. Clone or download this repo anywhere *outside* your own repo, e.g. `~/Downloads/design-system-workflow-kit`.
2. Open Claude Code in this folder and say: *"Install the kit into ~/code/our-repo"*. Claude
   surveys your repo, proposes where each file goes to fit your structure, and copies them
   once you agree.
3. Open Claude Code in your repo and say: *"Set up the workflow kit."*

You don't need to work out where anything goes yourself.

MIT licensed. See [LICENSE](LICENSE).
