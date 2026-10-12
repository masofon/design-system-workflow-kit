# Design system workflow: CLAUDE.md block

Paste everything below the line into your repo's `CLAUDE.md`, or let `ds-setup` do it. It holds
the standing rules every session should follow. The values behind them (branches, tracker,
roles, accessibility targets) live in `.claude/design-system.config.md`; setup is in `SETUP.md`.

---

## Design system workflow

- **The config is the source of system facts.** Read `.claude/design-system.config.md` for
  branches, the tracker, roles, paths and accessibility targets. Never guess a value it holds;
  if one is empty, ask.
- **Design facts** (Figma files, token tiers, modes, text styles, spacing) live in config § Paths
  → `design_foundations`. Read it before any Figma work.
- **Skills own the workflows.** CRDs, Figma components, tickets and the ready-for-dev hand-off
  each have a `ds-*` skill in `.claude/skills/`. Don't edit skills or their reference docs from
  inside other work; propose the change to config § Roles → `skill_owners`.

### Repo

- **Pull the integration branch at session start** if config § Repo → `pull_at_session_start` is
  Yes, fast-forward only.
- **Every change goes through a branch and a pull request** into config § Repo →
  `integration_branch`, docs and CRDs included. Never commit directly to the default or
  integration branch.
- **Every PR cites a ticket** (config § Tracker → `ticket_ref`) in its description and, if config
  says so, in the commit message. The `ds-ticket` skill raises the ticket if there isn't one.
- **Gloss every ticket or PR number** you mention (config § Writing → `gloss_ticket_ids`): a few
  words in brackets saying what it is, e.g. `#123 (Tooltip design)`. A bare number means nothing
  on sight, in chat, tickets, commits or handover notes.
- **A denied `git push` is usually the command's shape, not a missing permission.** Permission
  rules match the start of the command, so `cd repo && git push` or `git push | tail` can be
  refused when `git push` alone is allowed. Retry bare as `git -C /absolute/path push …` before
  concluding anything.

### Figma

- **Text content follows config § Writing → `text_case`** (sentence case by default). Never
  type ALL CAPS, Title Case or lowercase into the characters: capitalisation is presentation,
  applied by the text style's `textCase`. Exceptions: config § Writing → `case_exceptions`.
- **Every text node carries a text style.** No raw font, size or weight overrides. If styles
  bind their font to variables, the font follows the brand and theme; picking one by hand
  defeats that.
- **Auto-layout gap and padding bind to spacing tokens**, never raw numbers. Gap and padding can
  be separate scales that resolve differently at the same tier name: check
  `design_foundations` § Spacing before binding.
- **Deprecated tokens** follow config § Tokens → `deprecated_name`. Never bind anything new to
  one.
- **Downstream files see published values only.** After a token or library change, pause and ask
  config § Roles → `publisher` to publish before checking the result elsewhere.

### Token values

- **Check contrast before finishing any token value change**, against config § Accessibility →
  `standard`, for every combination in `contrast_must_pass`. Brands in `contrast_latitude` should
  be made passable, not blocked on.

### Reports and reviews

Follow config § Writing → `report_style`: concise, concerns first, lists over prose, and always
say what you deliberately did not change, and why.
