# Tracker adapters

`ds-ticket` holds the workflow. An adapter holds one tracker's mechanics: everything Claude
needs to carry out the workflow in *that* tracker without guessing. The team's choices (which
board, which stages, which tags) belong in config § Tracker, not here.

One file per tracker, named for it: `azure-boards.md`, `github.md`, `jira.md`, `trello.md`.

## What every adapter covers

Use these headings in this order, so a session can find the same thing in any adapter. If a
section doesn't apply, keep the heading and say why, e.g. "Trello has no item types".

0. **Setup choices** *(optional)*: only when the tracker can express the workflow more than one
   way (GitHub: sub-issues or milestones, a Projects board or labels). Each choice says how to
   work out the answer from the tracker, a recommendation, and which config key it fills.
   `ds-setup` runs it.

1. **Setup**: how a person connects Claude Code (MCP server or CLI), auth, and how to check it
   worked. Include the symptoms-and-causes table.
2. **Mapping the config**: how this tracker expresses each config concept:
   - **Board**: what config `board` must contain to address one board (org, project, repo,
     list…).
   - **Item roles**: native types, or labels, or issue types. How a container groups work
     items (parent link, sub-issues, epic link, checklist, milestone).
   - **Stages**: state, column, label or project field. Whether every role shares one stage
     set.
   - **Tags**: what they're called (tags, labels), how they're created, how casing behaves.
   - **Sprint or iteration**: whether the tracker has one, and how "backlog" is expressed.
3. **Reading**: the query that is truly scoped to the board, and any call that silently
   returns less than it looks like it does.
4. **Creating**: required fields per role, how parenting works (and whether it takes a second
   call), and bulk creation.
5. **Moving stages**: how, plus any stage-conditional fields (a field that only becomes
   required on one transition).
6. **Tags**: how to add and remove them, and any call that silently does nothing.
7. **Linking to PRs**: the ticket ref format, what the tracker links automatically from it,
   and how closing keywords behave against the team's branch model.
8. **Descriptions**: format (Markdown, HTML, wiki markup) and what gets escaped or mangled.
9. **Failure modes**: hangs, timeouts, partial writes, and what's safe to retry. Whether a
   timed-out create may have written anyway.
10. **Gotchas**: anything else learned the hard way. Date each one: it's an observation about
    someone else's product, and it can expire.

## Writing a new adapter

Ask Claude to draft one from this contract, the config and an existing adapter, then prove
each section against the live tracker before trusting it. In particular, **verify required
fields with the tracker's own schema call**, not from memory or docs.
