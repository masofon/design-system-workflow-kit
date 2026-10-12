# Adapter: GitHub Issues

Carries out `ds-ticket` through **GitHub Issues**, optionally with a **GitHub Projects** board,
using the `gh` CLI. Board, stages and tags come from config § Tracker.

GitHub gives a team several ways to express the same workflow concept, so **this adapter
starts with setup choices.** Make them once, record them in config § Tracker, and every
section below follows the recorded choice.

> **Unproven.** Drafted 2026-10-10 from documented GitHub behaviour and not yet run against a
> live repo. Items marked *(verify)* are the ones most likely to differ. Prove each one on the
> trial, then delete the marker and date the finding.

## 0. Setup choices

Ask Claude: *"Walk me through the GitHub tracker choices."* It **works out** each answer it can
from the repo first, shows the evidence, and asks only where the repo doesn't settle it. Then
it writes the confirmed answers into config § Tracker.

| # | Question | Options | Recommended | Work out from |
|---|---|---|---|---|
| 1 | **Which repo holds the issues?** | **A.** The code repo · **B.** A separate issues repo: ask for its URL | A, unless the team already triages elsewhere | `git remote get-url origin`; whether Issues is enabled |
| 2 | **Do you use milestones?** The container is always a **parent issue** with the work items as its **sub-issues**, so it can move across the board like an ADO Feature. A milestone is an optional extra on top | **A.** No · **B.** Yes, for releases or delivery dates: say how they're named | Whatever the team already does | Open milestones and their names; existing sub-issues |
| 3 | **How are item roles told apart?** | **A.** Org-level **issue types** (Feature, Task, Bug) · **B.** **Labels** (`type: container`, `type: work`, `type: bug`) | A if the org has issue types, otherwise B | Whether the owner is an org with issue types; existing `type:`-style labels |
| 4 | **How are stages expressed?** | **A.** A **Projects** board's single-select **Status** field · **B.** **Labels** (`stage: in review`…) | A: it's what GitHub teams look at. B if nobody will maintain a Project | `gh project list --owner {owner}`: a Project linked to the repo, and its Status options |
| 5 | **Where do PRs merge?** | **A.** The default branch (`main`) · **B.** An integration branch (`develop`) | Whatever the repo does; it decides question 6 | The default branch, and the base branch of recent merged PRs |
| 6 | **Do PRs use closing keywords?** | **A.** No: cite as `Refs #123` and move stages by hand · **B.** Yes, `Fixes #123` | **A.** Closing on merge skips `verify`, and with 5B it fires only at release | Recent merged PR bodies |
| 7 | **Enforce a ticket ref on every PR?** | **A.** Yes, with a small GitHub Action that fails a PR without `#\d+` · **B.** No, convention only | A if the team agrees: it makes the habit stick | Branch protection and rulesets: is a ticket check already required? |

How each answer lands in config:

- **1** → `board` (the `owner/repo`) and `ticket_ref` (`#{id}`, or `owner/repo#{id}` for 1B).
- **2** → § Tracker house rules (milestone use and naming). **3** → § Tracker item roles, *Expressed as* column.
- **4** → § Tracker stages, *Expressed as* column. For 4A, also the Project's owner and number in
  `board`.
- **5** → config § Repo → `integration_branch`.
- **6** → § Tracker house rules.
- **7** → § Tracker → `pr_needs_ticket`.

## 1. Setup

**`gh`, authenticated**, is all this adapter needs:

```bash
gh auth status
gh auth refresh -s project        # only for stages on a Projects board (choice 4A)
```

Projects commands fail without the `project` scope, which isn't granted by default.

*Alternative:* GitHub's official MCP server (`github/github-mcp-server`). It covers issues, but
Projects field edits and sub-issues are more direct through `gh`. Prefer `gh` unless the team
already runs the MCP.

### Check it worked

```bash
gh issue list --repo {owner/repo} --limit 5
gh project view {number} --owner {owner}        # choice 4A only
```

| Symptom | Cause |
|---|---|
| `gh: command not found`, or it dies at launch | Not installed, or the wrong CPU build |
| `HTTP 401` / not logged in | `gh auth login` |
| Projects command: `missing required scopes [project]` | `gh auth refresh -s project` |
| `Could not resolve to a Repository` | Wrong `owner/repo`, or no access |
| Issue create fails on a label | The label doesn't exist in that repo (see § 6) |

## 2. Mapping the config

- **Board**: `owner/repo` for issues, plus the Project's owner and number when stages live on a
  Project.
- **Item roles**: per choices 2 and 3.
  - **The container is a parent issue**, typed Feature where the org has issue types (3A), with
    the work items attached as its **sub-issues**. It moves through the stages like any item.
    If the org also has an Epic type, a container can itself sit under an Epic issue.
  - **A milestone** (2B) is an optional grouping on top, for a release or a delivery date. The
    parent issue and its sub-issues all carry it. It never replaces the parent issue: a
    milestone is only open or closed, so it can't sit in a stage.
  - **If sub-issues aren't available**, fall back to a task list of the work items in the
    parent issue's body. Say so at setup.
  - Issue types (3A) exist only for **organisation**-owned repos, and only if the org has
    defined them *(verify)*.
- **Stages**: per choice 4. **Issue state is only open or closed**, with a close reason of
  *completed* or *not planned*. That covers `done` (closed, completed) and `removed` (closed,
  not planned). Every other stage needs the Project Status field or a label.
- **Tags**: GitHub **labels**, which must exist in the repo before they can be applied. Keep stage
  labels (4B) and role labels (3B) visibly separate from the config's tags, e.g. a `stage:` or
  `type:` prefix.
- **Sprint**: none natively. A Project can carry an Iteration field, and some teams use milestones
  as sprints (2B). If neither is in use, "backlog" means no iteration or sprint milestone set.

## 3. Reading

```bash
gh issue list --repo {owner/repo} --state all --limit 500 \
  --json number,title,state,labels,assignees,author,milestone
gh issue view {n} --repo {owner/repo} --json number,title,body,state,labels,assignees,author,url
```

- **GitHub search matches whole words, not substrings.** For `ds-ticket`'s duplicate check,
  list issues as JSON and match titles locally; don't rely on `--search`.
- **Ownership** (`ds-ticket` rule 1): check `assignees` and `author.login` against
  `gh api user --jq .login`.
- **Project status** isn't on the issue. Read it from the Project:
  `gh project item-list {number} --owner {owner} --format json`.

## 4. Creating

```bash
gh issue create --repo {owner/repo} --title "…" --body-file body.md \
  --label "component" --assignee "{login}" [--milestone "…"]
```

- **Use `--body-file`**, not `--body`, to avoid shell-quoting damage to Markdown.
- **No required fields** beyond title, unless the repo has issue forms or templates. Forms don't
  bind `gh issue create`.
- **Sub-issues** take a second call. The REST endpoint wants the child's **database id**,
  not its issue number *(verify)*:
  ```bash
  child_id=$(gh api repos/{owner}/{repo}/issues/{child_n} --jq .id)
  gh api -X POST repos/{owner}/{repo}/issues/{parent_n}/sub_issues -F sub_issue_id=$child_id
  ```
- **Adding to the Project (4A)** is also a separate call:
  `gh project item-add {number} --owner {owner} --url {issue_url}`. It returns the item id,
  which the Status edit needs.
- **Issue type (3A)** isn't settable through every `gh` version *(verify)*. Fall back to
  `gh api` (REST `type` field or GraphQL) if `gh issue create` lacks a flag for it.
- **No dry run.** A create is live at once. Showing the ticket and getting approval is the
  safeguard.

## 5. Moving stages

**Project Status (4A).** Three ids are needed: the project, the Status field and the option.
Look them up once per session:

```bash
gh project view {number} --owner {owner} --format json --jq .id
gh project field-list {number} --owner {owner} --format json      # Status field id + option ids
gh project item-edit --project-id {pid} --id {item_id} \
  --field-id {status_field_id} --single-select-option-id {option_id}
```

**Labels (4B).** Remove the old stage label and add the new one in one call, so an item never
carries two stages:
`gh issue edit {n} --remove-label "stage: in progress" --add-label "stage: in review"`.

**`done` and `removed`** also close the issue:
`gh issue close {n} --reason completed` or `--reason "not planned"`.

**Projects built-in workflows can move items by themselves**: *Item closed → Done*, *Pull
request merged → Done* and others, some enabled by default on new Projects *(verify)*. Check the
Project's Workflows page during setup, and switch off any that would skip `verify`.

## 6. Tags

```bash
gh label list --repo {owner/repo}
gh label create "tokens" --repo {owner/repo} --description "…" --color "…"
gh issue edit {n} --add-label "tokens" --remove-label "…"
```

- **Labels must exist first.** Setup should create every label in config § Tracker tags (and
  any role or stage labels) once, with descriptions taken from the config.
- **Label names are case-insensitive and unique per repo** *(verify casing on apply)*. Match
  the existing label's casing exactly.

## 7. Linking to PRs

- **The ticket ref** is `#{id}` (same repo) or `owner/repo#{id}` (choice 1B).
- **Any `#{id}` in a PR body or commit adds a cross-reference** to the issue's timeline.
  There's nothing to add by hand.
- **Closing keywords** (`close`, `closes`, `closed`, `fix`, `fixes`, `fixed`, `resolve`,
  `resolves`, `resolved`) also link the PR in the issue's *Development* panel, and **close the
  issue when the PR merges into the default branch**. A PR into an integration branch doesn't
  close it. With choice 6A, write `Refs #{id}` and never use those words before a ref, even in
  passing prose.
- **Enforcing a ref (7A)**: GitHub has no built-in check. A minimal Action that greps the PR body
  for `#\d+` does it; draft one at setup if the team chooses 7A.

## 8. Descriptions

GitHub Flavoured Markdown, stored as written, with no escaping surprises. Task lists
(`- [ ]`) render as checkboxes. Config house rules decide whether to use them.

## 9. Failure modes

- **Secondary rate limits** trip on many creates in quick succession. Pause between bulk
  creates, and on `HTTP 403 … secondary rate limit` wait and retry the failed call only.
- **A failed `gh` call is explicit**: it exits non-zero with a message, so a create that errored
  normally hasn't written. Still list by title before retrying a create that timed out.
- **Claude Code's Auto mode can block external writes.** Hand the change to the person with the
  exact `gh` command and the issue URL, rather than working around it.

## 10. Gotchas

None proven yet. Add each one with its date as the trial finds it.
