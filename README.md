# Design system workflow kit

A Claude Code workflow for running a design system's component pipeline, from request to
hand-off:

- **A CRD per component.** One Component Reference Document holds what the component must do,
  how it's designed and how it's built. People and agents both read it.
- **Figma with gates.** A repeatable build process for the component library, with checks
  before anything is called ready.
- **Tickets that follow the work**, in whatever tracker you use.

Your system's specifics live in one config file. The skills hold the process.

**Start with [`SETUP.md`](SETUP.md).** In short:

1. Clone or download this repo anywhere *outside* your own repo, e.g. `~/Downloads/design-system-workflow-kit`.
2. Open Claude Code in this folder and say: *"Install the kit into ~/code/our-repo"*. Claude
   surveys your repo, proposes where each file goes to fit your structure, and copies them
   once you agree.
3. Open Claude Code in your repo and say: *"Set up the workflow kit."*

You don't need to work out where anything goes yourself.

MIT licensed. See [LICENSE](LICENSE).
