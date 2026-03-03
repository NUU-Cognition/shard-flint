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
  templates/
    tmp-f-flint_init-v0.1.md    # (System) Flint Init template
    tmp-f-concept-v0.1.md       # Concept note template
    tmp-f-record-v0.1.md        # Record note template
  knowledge/
    knw-f-workspace.md          # Workspace structure reference
    knw-f-templates.md          # Template syntax reference
    knw-f-artifacts.md          # Artifact conventions reference
    knw-f-cli.md                # CLI commands reference
  install/
    (System) Flint Init.md      # Default Flint Init for new workspaces
    otemp-f-default.md          # Obsidian template: bare note (UUID only)
    otmp-f-concept.md           # Obsidian template: concept note
    otmp-f-record.md            # Obsidian template: record note
```

## What This Shard Covers

- **Shard rules** — How to load, use, and discover shards (strict protocol)
- **Template rules** — How to read and generate from templates (strict protocol)
- **Agent environment** — Working directory, capabilities, sessions
- **Workspace structure** — Mesh, Shards, Types, Notes, Archive
- **Artifact conventions** — Frontmatter, tags, naming, session tracking
- **Notes** — Base notes, concept notes (`#note/concept`), record notes (`#note/record`)
- **Navigation** — Search-first, find files by name/tag
- **CLI commands** — Available tools and helpers
