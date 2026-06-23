# AWC Add-ons Installation Guide

## Manifest

Read the manifest from the same repo as this file:
`https://github.com/AmosChenZixuan/Agentic-working-contract/blob/main/manifest.yaml`

- `skills:` — list of `{name, description, upstream_guide}`. Cross-agent.
- `plugins:` — map of agent key → list of `{name, description}`. Match the current agent to a key **case-insensitively**, treating `claude`/`claude code`/`claude-code` as the same key (e.g. Claude Code → `claude`). If no key matches the current agent, skip plugins entirely.

## Flow

1. Drop already-installed entries (use your agent's native detection).
2. Present what's left to the user via your platform's multi-select Q&A. One group for `skills`, one for the current agent's matched `plugins` key. Skip a group if empty.
3. Install at **global/user scope by default** (e.g. `npx skills add … -g`, the global plugin path) — never project scope unless the user explicitly asks. The current directory is usually not their working project, and a local `.agent/` there is useless.
4. For each picked skill, fetch `upstream_guide` and follow it. For each picked plugin, install via your agent's native command.
5. Run installs serially in selection order.
6. Report installed / failed.
