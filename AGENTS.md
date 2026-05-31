# agent-marketplace

The plugin registry for the agent-plugins ecosystem. Metadata only — no plugin source, no binaries, no build pipeline live here. Claude Code (and other compatible agent CLIs) read this repo when a user runs `/plugin marketplace add Seretos/agent-marketplace`.

## Layout

```
.claude-plugin/marketplace.json   # the plugin registry, single source of truth
app-marketplace.json              # the app registry (downloadable apps), project root
.github/workflows/
  update-registry.yml             # processes plugin-release dispatches
  update-app-registry.yml         # processes app-release dispatches
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

### `icon` (optional, our own field)

`icon` is **not** part of Claude Code's documented plugin-entry schema — Claude Code and Codex silently ignore unrecognized fields, so it renders nowhere in either host's `/plugin` UI. It exists purely for **our own catalog page**, which reads `.claude-plugin/marketplace.json` and renders icons itself. Value is an opaque string the plugin chooses; recommended either a full `https://…` image URL or a path resolvable against the plugin's repo (e.g. `assets/icon.svg`).

It is **optional and rolled out gradually** — plugins adopt it one at a time. The dispatcher therefore:

- omits the `icon` key entirely when no icon is known (existing entries stay byte-for-byte unchanged), and
- **preserves** an icon already in the registry when a later release dispatches *without* one — a re-release from a not-yet-updated `release.yml` never wipes an icon. A non-empty payload icon always wins.

It is added **only to the Claude registry**, not `.agents/plugins/marketplace.json`, since the Codex parser's tolerance for unknown fields is unverified and the catalog reads the Claude file anyway.

### `description_url` and `tags` (optional, our own fields)

Two more optional, catalog-only fields that follow the **exact same rules as `icon`** (gradual rollout, preserve-on-omit, Claude registry only, ignored by Claude Code / Codex):

- `description_url` — a raw link to a longer-form `description.md` shipped on the plugin's `release` branch at the tagged commit (e.g. `https://raw.githubusercontent.com/${repo}/${ref}/description.md`). The field name is the same in the payload and the registry.
- `tags` — a JSON string array (e.g. `["mcp","windows"]`). The dispatch payload carries it as a JSON-encoded string; the dispatcher parses it and stores a real array. An empty/omitted value leaves the key out and preserves any existing tags.

## How entries get added

```
plugin repo (e.g. agent-vdesktop)
  release.yml after a successful build:
    POST /repos/Seretos/agent-marketplace/dispatches
    client_payload: { name, repo, version, category, description, icon?, description_url?, tags? }
      icon / description_url / tags are optional — omit them until the plugin has them
      (tags is a JSON string array, e.g. ["mcp","windows"])

this repo, update-registry.yml triggered by repository_dispatch:
  1. patches .claude-plugin/marketplace.json (upsert by plugin name)
     — builds the source object from repo + version
       (ref defaults to v{version}, overridable via client_payload.ref)
  2. force-pushes branch  plugin-update/{name}-v{version}
  3. opens PR if one isn't already open

human review + merge → entry is live
```

Manual PRs against `.claude-plugin/marketplace.json` are equally valid for hand-curated entries.

## app-marketplace.json schema

`app-marketplace.json` lives in the **project root** (not under `.claude-plugin/`) and is a separate registry for downloadable **apps**. It is structurally analogous to the plugin registry — same top-level `name` / `owner` / `metadata` — but holds an `apps` array, and app entries differ from plugins in two ways:

```json
{
  "name": "agent-some-app",
  "description": "...",
  "category": "...",
  "version": "0.1.0",
  "downloads": {
    "windows": "https://.../app-setup.exe",
    "macos":   "https://.../app.dmg",
    "linux":   "https://.../app.AppImage"
  }
}
```

- **No `source`.** Apps are not installed via a git source. Instead they carry `downloads`, a map of `platform -> url`. Only the platforms an app ships for are present.
- `description_url` and `icon` are the same optional, catalog-only fields as on plugins (preserve-on-omit, JSON-ignored by hosts).

## How apps get added

```
app repo
  release.yml after a successful build:
    POST /repos/Seretos/agent-marketplace/dispatches
    event_type: app-release
    client_payload: { name, repo, version, category, description, downloads, icon?, description_url? }
      downloads is a JSON object: { "windows": url, "macos": url, "linux": url }

this repo, update-app-registry.yml triggered by repository_dispatch:
  1. patches app-marketplace.json (upsert by app name)
  2. force-pushes branch  app-update/{name}-v{version}
  3. opens PR if one isn't already open
```

The app flow is deliberately separate from the plugin flow (own `event_type`, own workflow, own registry file) so the two never interfere. There is no Codex app registry.

## Cross-repo coupling

When the marketplace.json schema or dispatch payload changes, **every** plugin repo's `release.yml` and `dispatch.yml` need a matching update — the payload contract (`name`, `repo`, `version`, `category`, `description`, optional `icon`, `description_url`, `tags`) is shared. The plugin sends raw fields; this repo synthesizes the `source` object.

`icon`, `description_url` and `tags` are the **optional** fields in the contract: a plugin that doesn't send one loses nothing (the dispatcher preserves any existing value), so plugins can be migrated one at a time without coordinating a flag-day update across all repos.

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
