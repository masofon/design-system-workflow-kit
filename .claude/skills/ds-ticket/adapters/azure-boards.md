# Adapter: Azure Boards

Carries out `ds-ticket` through the **Azure DevOps MCP server**. Board coordinates, stages and
tags come from config § Tracker. Facts specific to one organisation's process template are
under **Instance notes** at the end; replace them with your own.

## 1. Setup

The adapter drives the Azure DevOps MCP server, which you configure once. Until then,
`ds-ticket` reports that it can't reach the tracker and stops.

Microsoft ships two servers. They differ in what they need from you, not in what they can do.

| | Remote | Local |
|---|---|---|
| Install | none | Node.js 20+ and `npx` |
| Transport | streamable HTTP | stdio |
| Auth | Microsoft Entra ID (OAuth) | Azure DevOps PAT, or Entra |
| Claude Code needs | a **Microsoft Entra app registration**, one-off, with tenant admin consent | nothing beyond Node |

**Remote suits a team**: one admin action, then everyone runs one command and installs nothing.
**Local is faster for one person**, if they have Node.

### Remote, one-off, by someone with Entra admin rights

1. Check the **Azure DevOps MCP** enterprise application is provisioned in the tenant.
2. Microsoft Entra admin center → **App registrations** → new registration.
3. **Authentication** → **Add redirect URI** → **Mobile and desktop applications** →
   `http://localhost:3118/callback`.
4. On the same page → **Settings** → enable **Allow public client flows**.
5. **API permissions** → **Add a permission** → **APIs my organization uses** → search
   `Azure DevOps MCP` (app ID `2a72489c-aab2-4b65-b93a-a91edccf33b8`) → add the delegated
   permissions → **Grant admin consent**.
6. Copy the **Application (client) ID** and share it with the team. It's the same for
   everyone.

### Remote, per person

```bash
claude mcp add --transport http ado https://mcp.dev.azure.com/{organization} \
  --client-id {client-id} --callback-port 3118
```

Then start `claude`, run `/mcp`, and complete the browser sign-in.

### Local

Needs **Node.js 20+**. Check with `node --version`; if it says "command not found", this
route is closed until you install Node.

```bash
claude mcp add --scope user --transport stdio ado -- \
  npx -y @azure-devops/mcp {organization} -d core work-items work
```

`--scope user` registers the server for every directory, so a session outside the design
system repo can still raise a ticket. Interactive browser sign-in is the default.
`--authentication pat` uses a Personal Access Token instead (**Work Items → Read, write, &
manage**).

### Trimming the tool list

Both servers load every Azure DevOps tool by default: repos, pipelines, wikis, test plans.
This adapter needs only work items, iterations and projects.

- **Remote**: set the `X-MCP-Toolsets` header to `core,wit,work`.
- **Local**: `-d` takes a space-separated list, not repeated flags. The server echoes
  `enabledDomains` on startup, so you can check it took.

**Trim only once it works**: a failure with a trimmed list is ambiguous. The change lands on the
next Claude Code restart.

### Check it worked

Run `claude mcp list`, then ask Claude for your assigned items **on the board**. A plain
"assigned to me" query is project-wide and proves only auth and org; a WIQL filtered on the
area path proves the board too.

| Symptom | Cause |
|---|---|
| The skill says the tracker isn't available | Not configured, or `claude` needs restarting |
| Authentication fails | Entra credentials, or no access to the org. Run config `auth_fix` |
| Connection refused | Outbound HTTPS to `mcp.dev.azure.com` blocked: proxy or VPN |
| Connects but returns nothing | Permissions on the project, not a setup fault |
| `npx: command not found` | No Node: use the remote server |

**Also needed:** an authenticated `gh` (or your repo host's CLI) for putting the ticket ref into
a PR description.

**Sources:**
[overview](https://learn.microsoft.com/en-us/azure/devops/mcp-server/mcp-server-overview?view=azure-devops) ·
[remote setup](https://learn.microsoft.com/en-us/azure/devops/mcp-server/remote-mcp-server?view=azure-devops) ·
[microsoft/azure-devops-mcp](https://github.com/microsoft/azure-devops-mcp)

## 2. Mapping the config

- **Board**: organization, project, team and **area path**. A team's board is usually scoped
  by area path, not by iteration, so "the backlog" means that area path at the project's
  root iteration.
- **Item roles**: native work item types. Containers group work items through a **parent**
  link. The conventional nesting is Epic → Feature → Story/Bug → Task, but live boards rarely
  follow it strictly. Don't "fix" someone else's placement.
- **Stages**: the `System.State` field. **Every type has its own state set.** A Feature may
  lack states a User Story has, and a Task often has only four. Don't assume one role's list
  applies to another.
- **Tags**: free-text `System.Tags`, matched case-insensitively. **The existing definition's
  casing wins**, so sending `documentation` silently joins items already tagged
  `Documentation`.
- **Sprint**: `System.IterationPath`. Set area path and iteration explicitly on every create.
  The current sprint is a separate axis from the backlog.

## 3. Reading

**Reading is always two calls.** `wit_query` **ignores the SELECT column list**, and
`wit_work_item action: my` does the same: both return bare ids. Query for ids, then call
`wit_work_item action: get_batch` with an explicit `fields` array to get titles, states and
tags.

**`action: my` does not prove the board.** It is project-wide. Only a WIQL filtered on
`[System.AreaPath]` is scoped to the board.

## 4. Creating

- **Required fields differ per type**, and ADO rejects a create that omits any of them.
  **`wit_work_item action: get_type` returns every field with an `alwaysRequired` flag.** Use
  it rather than trusting any table, including the one under Instance notes.
- **`System.Parent` in a create is silently dropped**: no error, and the field is simply absent
  from the response. Parent the item with a second call, `wit_work_item_link_write action:
  link` with `type: parent`.
- **`add_child` parents in one call.** `wit_work_item_write action: add_child` creates several
  work items under a container at once, already parented. It takes only title, description,
  area and iteration, so assignee, state and tags follow in one `update_batch` call.
- **There is no dry run.** The MCP has no validate-only mode, so a bad field set is discovered
  by the create that writes it. Showing the ticket and getting approval is the only safeguard.

## 5. Moving stages

Set `System.State`. **Some fields become required only on one transition.** They aren't on
the always-required list, and ADO answers `TF401320 ... Required, InvalidEmpty`. Expect them
on `Closed` in particular.

**Built-in automation won't keep stages in sync.** Azure Boards can move an item from a
`Fixes AB#123` keyword, but only on merge to the repo's *default* branch, and it **closes** the
item, skipping `verify`. If PRs merge into an integration branch, it fires only at release, in
bulk. Move stages by hand, per `ds-ticket`.

## 6. Tags

**Removing a tag needs a `replace` operation, not `add`.** An `add` returns HTTP 200 with the
revision unchanged: it silently does nothing.

## 7. Linking to PRs

The ticket ref is `AB#{id}`. If the repo is on GitHub with the Azure Boards app installed, put
the ref in the **commit message** as well as the PR body, and the integration attaches both a
`GitHub Commit` and a `GitHub Pull Request` artifact link to the item by itself. **Don't add a
reverse link by hand**; there is nothing to add. *(Azure Repos links work items through its own
PR UI; record what yours does under Instance notes.)*

Any `AB#{id}` *anywhere* in a PR body gets linked, so keep passing references to other items
out of it.

## 8. Descriptions

Markdown. ADO HTML-escapes `<`, `>`, `&` and straight double quotes on write, and Markdown
decodes them again everywhere *except* inside code spans, so `` `Array<T>` `` renders as
`Array&lt;T&gt;`. Prefer → over `->`, use curly quotes “like this” rather than straight
ones, and keep `<`, `>` and `&` out of backticks.

## 9. Failure modes

- **A call that hangs is usually waiting on a sign-in window, and retrying works.** The MCP
  can pop a browser auth prompt per call that reports failure even though the person is signed
  in, and the call then sits until it times out. Stop it and retry, repeatedly if needed.
  Reads and link or field updates are safe to retry as-is. *(2026-09)*
- **A create that times out may have written nothing, or something.** **Query the board for
  the titles before retrying**; never re-create blind. Stop any stale background write once
  you have redone it by hand, because an `unlink` landing late silently undoes the link you made
  after it. *(2026-09)*
- **Claude Code's Auto mode can block writes** as an *External System Write*. It's
  intermittent, and not a permission or auth fault. Follow `ds-ticket`'s classifier rule
  (smaller pieces once, then hand over), or suggest a session not in Auto mode. *(2026-09)*

## 10. Gotchas

- `visualstudio.com` and `dev.azure.com` are the same org. The API answers on
  `dev.azure.com`; human-facing links use `visualstudio.com`.

## Instance notes *(fill in for your organisation)*

Process templates vary between organisations, and custom fields are common. Record what
`get_type` and your first live runs show, so later sessions don't rediscover it.

| Type | Also required | Priority |
|---|---|---|
| | | |

- **States per type**, in board order.
- **Stage-conditional fields**: any field required on one transition only (often `Closed`),
  and the value to use.
- **Custom fields** whose reference names aren't guessable (`Custom.…`).
- **Tags to ignore**: artefacts of a migration or another team's process.
