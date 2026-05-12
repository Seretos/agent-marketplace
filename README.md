# agent-marketplace

Central registry for the agent plugin platform. Each entry in `.claude-plugin/marketplace.json` points at a plugin repository (MCP server, skill, command set, hooks, or any mix). Claude Code reads this registry to resolve and install plugins.

## How plugins get added

The normal path is fully automated end-to-end:

1. A plugin repo (e.g. `Seretos/agent-vdesktop`) produces a tagged release on its own `release` branch.
2. Its release pipeline fires a `repository_dispatch` event of type `plugin-release` at this repo, carrying the plugin's name, version, source, category, and description.
3. `.github/workflows/update-registry.yml` consumes the event, patches `.claude-plugin/marketplace.json`, opens a PR on a `plugin-update/{name}-v{version}` branch.
4. A human reviews and merges the PR.
5. `.github/workflows/tag-on-merge.yml` parses the branch name and pushes the marketplace tag `{plugin-name}--v{version}` onto the merge commit.

Manual PRs against `.claude-plugin/marketplace.json` are also fine for hand-curated entries — as long as the branch follows the `plugin-update/{name}-v{version}` convention, the tag-on-merge workflow will set the same tag.

## Tag scheme

Marketplace tags use the form:

```
{plugin-name}--v{version}
```

Examples: `agent-vdesktop--v0.1.0`, `agent-github--v2.3.1`. The double dash separates the plugin name (which may itself contain single dashes) from the `v`-prefixed semver. Claude Code uses these tags to pin a specific `marketplace.json` revision per plugin version during dependency resolution.

## Repo layout

```
.claude-plugin/marketplace.json    central registry, single source of truth
.github/workflows/                 update-registry.yml, tag-on-merge.yml
```

Nothing else lives here — no plugin source code, no build artifacts. This repo is metadata only.
