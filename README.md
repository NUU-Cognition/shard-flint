# Flint (Core Shard)

The foundational shard for all Flint workspaces. Defines how agents operate inside a Flint — navigating the workspace, loading and using shards, reading templates, managing sessions, and working with artifacts.

Every Flint workspace requires this shard. It provides the base conventions that all other shards build upon.

## Structure

```
Shards/(Dev) Flint/
  shard.yaml                    # Manifest
  init-f.md                     # Init file — agent environment context
  skills/
    sk-f-create_note.md         # Create evergreen concept notes
    sk-f-init_update.md         # Update Flint Init from session
    sk-f-init_full_update.md    # Full research & rewrite of Flint Init
    sk-f-upgrade.md             # Upgrade Flint to latest version
  templates/
    tmp-f-note.md               # Concept note template
    tmp-f-flint_init.md         # (System) Flint Init template
  knowledge/
    knw-f-workspace.md          # Workspace structure reference
    knw-f-templates.md          # Template syntax reference
    knw-f-artifacts.md          # Artifact conventions reference
  install/
    (System) Flint Init.md      # Default Flint Init for new workspaces
```

## What This Shard Covers

- **Agent environment** — What agents can do, how sessions work
- **Workspace structure** — Mesh, Shards, Types, Notes, Archive
- **Shard system** — Loading, using, discovering shards
- **Template syntax** — How to read and generate from templates
- **Artifact conventions** — Frontmatter, tags, status, naming, session tracking
- **Concept notes** — Evergreen atomic knowledge notes
- **Navigation** — How to find things in a Flint workspace
- **CLI commands** — Available tools and helpers
