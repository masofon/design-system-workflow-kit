# ds-ticket — setup

`SKILL.md` is what Claude reads. This is how a person turns it on.

1. Fill in `.claude/design-system.config.md` § Tracker: which tracker, the board, item roles,
   stages, spin-up titles, tags and house rules.
2. Open the adapter for your tracker in `adapters/` and follow its **Setup** section to connect
   Claude Code to it.
3. No adapter for your tracker yet? `adapters/README.md` is the contract every adapter fills.
   Ask Claude to draft one, then prove it against the live tracker.

| Tracker | Adapter |
|---|---|
| Azure Boards | `adapters/azure-boards.md` |
| GitHub Issues | `adapters/github.md`. Starts with setup choices; unproven until the first trial |
