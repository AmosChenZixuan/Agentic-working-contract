# AWC Add-ons Installation Guide

## Manifest

Read the manifest from the same repo as this file:
`https://github.com/AmosChenZixuan/Agentic-working-contract/blob/main/manifest.yaml`

- `skills:` — list of packages `{install_guide, provides:[{name, description}]}`. Cross-agent. Each package's `provides` are the skills its one `install_guide` installs.
- `plugins:` — map of agent key → list of `{name, description}`. Match the current agent to a key **case-insensitively**, treating `claude`/`claude code`/`claude-code` as the same key (e.g. Claude Code → `claude`). If no key matches the current agent, skip plugins entirely.

**Valid shape:**

```yaml
skills:
  - install_guide: https://github.com/owner/repo/blob/main/README.md
    provides:
      - name: skill-a
        description: what skill-a does
      - name: skill-b
        description: what skill-b does
plugins:
  claude:
    - name: plugin-name
      description: what the plugin does
```

Every entry under `skills:` must have both `install_guide` and `provides`. Entries missing either are malformed — infer the parent package from context (closest preceding valid package block), don't ask.

## Flow

1. Drop already-installed entries (use your agent's native detection).
2. Present what's left to the user via your platform's multi-select Q&A. List the individual skills (flatten each package's `provides`) in one group, and the current agent's matched `plugins` key in another. Skip a group if empty.
3. Install at **global/user scope by default** (e.g. `npx skills add … -g`, the global plugin path) — never project scope unless the user explicitly asks. The current directory is usually not their working project, and a local `.agent/` there is useless.
4. Group the picked skills by their package's `install_guide`; fetch each guide **once** and follow it to install every picked skill from that package. For each picked plugin, install via your agent's native command.
5. Run installs serially in selection order.
6. Report installed / failed.
