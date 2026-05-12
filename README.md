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

The full list of plugins is in [`.claude-plugin/marketplace.json`](.claude-plugin/marketplace.json). Each entry points at its own repository, where the plugin's own README explains what it does and how to use it.

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
