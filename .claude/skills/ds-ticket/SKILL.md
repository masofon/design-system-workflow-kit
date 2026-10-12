---
name: ds-ticket
description: Create and manage the design system's tickets in whatever tracker the team uses — Azure Boards, GitHub Issues, Jira, Trello or another — and link them to pull requests. Use when a PR needs a ticket reference; when asked to raise, make, create, file, log or open a ticket, issue, story, feature, epic, task, card or bug for the design system; when reporting a defect in a design system component; when spinning up the tickets for a new component, or for an amendment to a shipped one; when someone building a stable component cannot author in Figma and needs the design system team to design it from the CRD; when picking up an existing ticket; when backfilling ticket links onto existing PRs; or when moving a ticket to another stage.
---

# Design system tickets

This skill holds the workflow's ticket conventions. The tracker's details come from two places:

- **`.claude/design-system.config.md` § Tracker**: which tracker, how to reach it, the board,
  item roles, stages, spin-up titles, tags and house rules. Every *config §* reference below
  points there.
- **The adapter** named in config § Tracker → `adapter`, in `adapters/` beside this file:
  access setup and the tracker's mechanics (field names, query patterns, API quirks). **Read
  it before your first tracker call.**

**Tracker unreachable?** Say so and point at the adapter's setup section. Don't fall back to
raw API calls, and never ask for a personal access token in chat.

**A tracker write blocked by Claude Code's permission classifier?** (Not a tracker error: the
client refused before the call went out.) Retry once in smaller pieces: one item per call
instead of a batch, or one field per update. If that's blocked too, stop and hand the change to
the person: the item URL, and the field and value or the stage. Don't work around the block any
other way. The adapter's § Failure modes covers the tracker's own errors.

**No adapter for the configured tracker?** Say so, and offer to draft one from
`adapters/README.md` (the contract every adapter fills) and the existing adapters. Don't
improvise tracker calls without one.

## Vocabulary

Config § Tracker item roles and § Tracker stages map each term to the tracker's own:

| Term | Means |
|---|---|
| **Container** | Groups the work for one component or one amendment |
| **Work item** | One unit of work: one PR, one person |
| **Defect** | A bug in a shipped component |
| **Stage** | Where an item is: `new`, `ready_for_dev`, `in_progress`, `in_review`, `verify`, `done`, `blocked`, `removed` |
| **Tag** | A label from the allowed list in config § Tracker tags |
| **Ticket ref** | How a commit or PR cites an item: config § Tracker → `ticket_ref` |

## Two rules, every time

### 1. Never modify an item that isn't yours

Before any edit, stage move, link or reparent, read the item and check its assignee and
creator against the current user. If it belongs to someone else, **stop and name the owner.**
Proceed only if the person named that specific item when asking for the change. Never on your
own initiative, and never to tidy the board.

### 2. Check for duplicates before every create

Search the board for near-identical titles first.

- Search on the title's **distinctive** words, matching **substrings, not whole words**.
  `Basic_Card` is one word to most search engines, so word matching misses `Basic Card`.
- **The verb is the distinction.** `Design X`, `Develop X` and `Write the X CRD` are different
  items. Strip `component`, `add` and `new` when comparing; never strip the verb.
- **Identical** once case and punctuation are folded: don't create it. Show the existing item
  and ask.
- **Similar**: create it, but say what it resembles.
- Role matters: a container named `Component: Accordion` doesn't clash with a work item named
  `Design the Accordion component`.

Prevent new duplicates; don't report existing ones.

## Writing the ticket

- **Title**: sentence case, describing the outcome. No `feat:` or `fix:` prefix, no ticket-id
  prefix.
- **Description**: a context paragraph (what's missing today), then what changes.
- **Where a CRD is the source of truth, link it; don't restate it.** Component items get a line
  or two plus the CRD link: a description that duplicates the CRD goes unread and stale.
  Process and workflow items get a real description.
- Apply config § Tracker house rules: acceptance criteria, assignment, any closing line.
- **Assign by who did the work.** Infer the owner from who actually worked on it (the CRD
  author, the Figma page author, the existing items), never from who *asked*, and propose it
  with the rest of the ticket. Leave develop items unassigned unless a developer is already on
  them.
- **Always hand back the item URL** (config § Tracker → `item_url`), leading with it, after
  every change: a create, a stage move, an edit, a link.

The adapter covers how the tracker renders descriptions and what it escapes.

## Tags

Suggest from config § Tracker tags, **confirm, then apply**. Never auto-apply, never invent a
tag, and honour the never-use and casing rules there.

**Don't copy the tags on existing items.** Older items often carry migration artefacts and
one-off labels; the config list is the taxonomy.

## Ask, don't guess

Use **AskUserQuestion**, not a question buried in prose, and ask about several things at once.

| Setting | Ask when |
|---|---|
| Role | the request said "ticket" without saying which |
| Parent | creating a work item: list real containers, don't invent them |
| Sprint or milestone | the tracker has them and placement is unclear |
| Stage | the work already exists: `new`, `in_progress` or `in_review` |
| Tags | always: propose, then confirm |
| Severity, found-in | creating a defect |

Don't ask about anything the config or the adapter fixes to one value.

**Show the ticket and get approval before creating it.** A create is visible to the whole team
at once, and most trackers have no clean delete; `removed` is the usual way out.

## Spinning up a component's items

On explicit human command, for the components the person names, one at a time or as a list.
Titles come from config § Tracker spin-up → `component`: a container, then the work items in
order (CRD, design, develop).

**Check what already exists first**: the board, the CRD folder (`lifecycle` and `status` in
front matter) and the component's page in the Figma library, so the items reflect where the
work really is rather than all starting at `new`.

- **CRD already written**: create the CRD item anyway and move it to `done`, so the container
  shows the whole lifecycle. A CRD in progress goes to `in_progress`. If the person already
  has an older item for writing this CRD, reparent it rather than duplicating it.
- **Design item at `in_progress` when real work exists in Figma**, `new` when there is none.
- **Design-only components** (needed for designer use, not being built) get the container and
  the design item only, and the container says so in its first line.
- **Bodies are a pointer**: a line or two, then the CRD link, or the Figma page when there is
  no CRD yet. **The develop item carries the CRD link from creation**, never "link to follow".
  No CRD yet: say so, and ask before creating anything.
- If the container already exists, **don't create a second one**. Bring it up to date instead,
  within rule 1.
- Never pick the components yourself, and never sweep the backlog. A CRD at
  `lifecycle: backlog` is not proof the team has decided to start it.
- New items go where config § Tracker → `new_items_go_to` says, never straight into a sprint.

## Spinning up an amendment's items

Only if config § CRDs → `amendments` is Yes. For a change to a shipped component, following
config § Paths → `crd_how_to` § Amending a shipped component. On explicit human command, one
named amendment at a time. Titles come from config § Tracker spin-up → `amendment`.

- **One container per amendment.** An amendment can cover several changes; `{summary}` names
  it as a whole. Take `{n}` from the CRD.
- **Check whether the amendment is written.** Look for the `# Amendment {n}` block in the CRD
  on the integration branch and in open PRs touching it, and for a request item on the board.
  Report what you find and ask. If it's written, there's no Write item.
- **Check the behaviour isn't already built.**
- **The order is Write → Design → Fold → Develop.** The build works from the folded contract.
- **Raise the items only.** Whoever picks up the Write or Fold item does the CRD work, in their
  own session, with `ds-crd-update`.
- **Everything unassigned and in the backlog** unless the requester says otherwise.
- **Link the container** as related to the original build's container and to the request item,
  if the tracker supports links.
- **Bodies are a pointer**: a line or two and the CRD link, the develop item included.

## Picking up an item that already exists

When someone starts on an item already on the board, spun up earlier or raised for the team to
claim:

1. **Find it**, by id or by a title search under its container. Don't create a new one because
   the search was lazy.
2. **Read it and apply rule 1.** Unassigned or already theirs: carry on. Assigned to someone
   else: stop and name them.
3. **Assign it to whoever is doing the work and move it to `in_progress`.** Nothing else: leave
   sprint placement and the description alone unless asked.
4. Hand back the URL.

## Design request: a stable component with no Figma seat

Only if config § CRDs → `stable_components` is Yes. This is where `ds-design-component` sends a
contributor who has built a stable component but can't author the Figma side. Without it the
handoff dead-ends, and the design system team never learns the stable component exists.

Create a **work item**, `Design the stable {Name} component`, at `new`, **unassigned**, in the
backlog, for the design system team to claim. The body is a pointer: who needs it and why, the
CRD link, and the stable build link if there is one.

Unassigned is deliberate: it stays editable by whoever created it.

### Finding the CRD

The person asking usually won't know the exact filename, and often has no clone because
they're working in their own repo. Both are normal.

CRDs live in config § Paths → `crd_dir`. **Don't rely on the filename pattern.** Skip the
template, the how-to and the README; the template's `lifecycle:` lists the whole enum, not one
value.

1. With a clone, match the component name against each CRD's `name:` front matter.
2. Without one, list the folder through the repo host's API and match on filename.
3. Match loosely: case, underscores, spaces and a missing suffix all vary. If more than one
   hit is plausible, **show the candidates and ask**.
4. **If nothing resolves, don't file an item with no reference.** Ask for the URL, or confirm
   that no CRD exists yet.

## Linking an item to a PR

Put the ticket ref (config § Tracker → `ticket_ref`) in the **PR description**, and in the
**commit message** too if config § Tracker → `ticket_ref_in_commit` is Yes. Leave a body that
already carries a ref alone.

**Check the adapter before adding a reverse link** from the item to the PR. Many trackers
create it automatically from the ref, and a hand-made one duplicates it.

**Beware closing keywords.** Some trackers close an item when a PR saying `Fixes #123` or
`Closes #123` merges, often only on merge to the *default* branch. That skips `verify`, and if
the team merges into an integration branch, the close fires late or in bulk. The adapter says
whether this applies; when in doubt, cite the ref without a keyword.

## Keeping stages in sync with PRs

Assume nothing does this automatically unless the adapter says otherwise. Move items by hand:

| PR | Item goes to |
|---|---|
| Open, draft | `in_progress` |
| Open, ready for review | `in_review` |
| **Merged** | **`verify`**: merged means ready to verify, not done |
| Closed unmerged | decide by hand |

Apply config § Tracker stages: forward only, and never move an item out of the stages listed
there.

**The merge move is the one that gets missed**, because the PR usually merges after the session
that opened it has moved on. When the person says a PR merged, or you find it merged, move its
item to `verify` then.

**Only look at the person's own PRs.** An item of theirs whose PR a colleague raised is not
yours to fix; colleagues move their own items when they do the work.

**Hand-off to dev** moves the container and the develop item to `ready_for_dev`. That move
belongs to `ds-ready-for-dev`, after it has checked the hand-off. Dev review is the first step
of the develop item, not a precondition for the move.
