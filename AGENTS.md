# agent-marketplace

The plugin registry for the agent-plugins ecosystem. Metadata only — no plugin source, no binaries, no build pipeline live here. Claude Code (and other compatible agent CLIs) read this repo when a user runs `/plugin marketplace add Seretos/agent-marketplace`.

## Layout

```
.claude-plugin/marketplace.json   # the registry, single source of truth
.github/workflows/
  update-registry.yml             # processes plugin-release dispatches
README.md                         # user-facing
AGENTS.md                         # this file
.gitignore
```

## marketplace.json schema

Each plugin entry uses Claude Code's **object** source format:

```json
{
  "name": "agent-vdesktop",
  "description": "...",
  "source": {
    "source": "github",
    "repo":   "Seretos/agent-vdesktop",
    "ref":    "v0.0.1"
  },
  "category": "mcp",
  "version": "0.0.1"
}
```

- `source.ref` points at the plugin's git tag (which itself points at a release-branch commit with install-ready files).
- `version` and `source.ref` are kept in lockstep by the dispatcher (`ref = v{version}`).
- A **bare string** source (e.g. `"source": "Seretos/agent-vdesktop"`) is **not** accepted by Claude Code — it interprets strings as local relative paths only. The object form is required for any remote git plugin.

Top-level required fields: `name`, `owner` (with `owner.name`), `plugins` (array). `metadata.version` / `metadata.description` are optional.

## How entries get added

```
plugin repo (e.g. agent-vdesktop)
  release.yml after a successful build:
    POST /repos/Seretos/agent-marketplace/dispatches
    client_payload: { name, repo, version, category, description }

this repo, update-registry.yml triggered by repository_dispatch:
  1. patches .claude-plugin/marketplace.json (upsert by plugin name)
     — builds the source object from repo + version
       (ref defaults to v{version}, overridable via client_payload.ref)
  2. force-pushes branch  plugin-update/{name}-v{version}
  3. opens PR if one isn't already open

human review + merge → entry is live
```

Manual PRs against `.claude-plugin/marketplace.json` are equally valid for hand-curated entries.

## Cross-repo coupling

When the marketplace.json schema or dispatch payload changes, **every** plugin repo's `release.yml` and `dispatch.yml` need a matching update — the payload contract (`name`, `repo`, `version`, `category`, `description`) is shared. The plugin sends raw fields; this repo synthesizes the `source` object.

## Things that turned out NOT to be needed

- **`{plugin-name}--v{version}` tags on this repo.** Early iterations had a `tag-on-merge.yml` workflow that pushed these. Claude Code resolves versions from the marketplace.json content itself, so the tags were duplicate state. Removed.
- **`release: published` triggers.** Tried, didn't fire reliably for releases targeted at non-default branches. Plugin repos now dispatch via direct POST.

## GitHub repo settings that matter

- Actions → General → Workflow permissions → "Allow GitHub Actions to create and approve pull requests" must be enabled, otherwise `gh pr create` in update-registry.yml fails.
- `update-registry.yml` declares `permissions.pull-requests: write` and `permissions.contents: write` at the job level.

## Conventions

- Force-push the plugin-update branch on re-runs (idempotent).
- Skip `gh pr create` if a PR for the branch already exists; force-push refreshes its head.
- Don't push direct to main from workflows — always go via PR for the human review gate.
