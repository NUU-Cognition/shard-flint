# Flint (Core Shard)

The foundational shard for all Flint workspaces. Defines the rules agents must follow — how to load and use shards, how to read templates, workspace conventions, and note types.

Every Flint workspace requires this shard. It provides the base conventions that all other shards build upon.

## Structure

```
Shards/(Dev) Flint/
  shard.yaml                    # Manifest
  init-f.md                     # Init — shard rules, template rules, required reading
  skills/
    sk-f-init_update.md         # Update Flint Init from session changes
    sk-f-init_full_update.md    # Full research & rewrite of Flint Init
    sk-f-shard_browse.md        # Browse the shard catalog, suggest shards for the Flint's purpose, install picks
  templates/
    tmp-f-flint_init-v0.1.md    # (System) Flint Init template
  knowledge/
    knw-f-workspace.md          # Workspace structure reference
    knw-f-templates.md          # Template syntax reference
    knw-f-artifacts.md          # Artifact conventions reference
    knw-f-cli.md                # CLI commands reference
  install/
    (System) Flint Init.md      # Default Flint Init for new workspaces
    otemp-f-default.md          # Obsidian template: bare note (UUID only)
```

## What This Shard Covers

- **Shard rules** — How to load, use, and discover shards (strict protocol)
- **Choosing shards** — look before you install or create a shard: `flint shard list` and the registry site (`flint shard browse` is planned; it is not in this build of the CLI)
- **Template rules** — How to read and generate from templates (strict protocol)
- **Agent environment** — Working directory, capabilities, sessions
- **Workspace structure** — Mesh, Shards, Types, Notes, Archive
- **Artifact conventions** — Frontmatter, tags, naming, session tracking
- **Notes** — Base notes, concept notes (`#note/concept`), record notes (`#note/record`)
- **Navigation** — Search-first, find files by name/tag
- **CLI commands** — Available tools and helpers
