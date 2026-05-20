# agent-marketplace

A curated registry of plugins for AI coding agents — MCP servers, skills, slash commands, hooks, or any mix. All plugins listed here are maintained by [@Seretos](https://github.com/Seretos).

Designed for **Claude Code** and other agent CLIs with compatible plugin systems.

## Quick install

Add this marketplace to your agent, then pull whatever plugin you need from it.

**Claude Code:**

```
/plugin marketplace add Seretos/agent-marketplace
/plugin install <plugin-name>@agent-marketplace
```

That's it. The agent fetches the plugin from its own repo at the version pinned here.

## What's inside

The full list of plugins is in [`.claude-plugin/marketplace.json`](.claude-plugin/marketplace.json) (Claude Code) and [`.agents/plugins/marketplace.json`](.agents/plugins/marketplace.json) (Codex). Both registries describe the same plugins in the format each host expects — release-pipeline CI keeps them in sync. Each entry points at its own repository, where the plugin's own README explains what it does and how to use it.

## Codex support

Codex support is **best-effort and currently untested by us** — install at your own risk. The `.agents/plugins/marketplace.json` registry is kept in sync with the Claude one and the per-plugin `.codex-plugin/plugin.json` manifests are doc-aligned (`${PLUGIN_ROOT}` placeholder per the [Codex plugin docs](https://developers.openai.com/codex/plugins/build)).

Per-plugin status:

- **agent-vdesktop** — Windows-only by design (Microsoft Virtual Desktop APIs). Codex/Windows should work; Linux not applicable.
- **agent-vdesktop-skill** — skill-only, no MCP server. Should work on any Codex platform.
- **agent-worktree**, **agent-project-issues** — ship per-OS binaries (`bin/<name>` Linux ELF + `bin/<name>.exe` Windows PE) in the same directory. **Codex/Linux should work; Codex/Windows is known-broken** because Codex does not resolve extensionless absolute paths via PATHEXT — see [openai/codex#16229](https://github.com/openai/codex/issues/16229). Waiting for an upstream fix; no workaround on our side.

If you hit issues outside these known cases, please file against the affected plugin's repo.

## Alternative installs

If your agent doesn't support marketplaces yet, you can install any plugin from this registry **directly** from its own repo — the marketplace is just a convenience layer over individual plugin releases.

1. Open `.claude-plugin/marketplace.json` and find the plugin entry.
2. Note the `source.repo` and `source.ref` (e.g. `Seretos/agent-vdesktop` at `v0.0.1`).
3. Follow the install instructions in that plugin's README.

For a one-off install of a specific version without touching the marketplace:

```
/plugin install <owner>/<repo>@<ref>
```

(Claude Code accepts a `owner/repo@ref` shorthand for ad-hoc installs.)

## How plugins get added

Plugin repos publish via GitHub `repository_dispatch` → CI opens a PR against this repo → I review and merge → the registry entry is live. Hand-curated PRs are also welcome.
