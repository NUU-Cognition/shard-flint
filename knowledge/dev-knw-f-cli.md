---
description: "Flint CLI commands for agent workflows"
---

# Knowledge: CLI Reference

Quick reference for Flint CLI commands relevant to agent workflows.

## Artifact Helpers

```bash
flint helper type newnumber <Type>    # Next number for a typed artifact (zero-padded, 3 digits)
flint helper type newnumber Task      # → 119
flint helper type newnumber Notepad   # → 034
```

## Identity

```bash
flint whoami "Name"                      # Set person identity → .flint/identity.json + Mesh/People/@Name.md
flint whoami                             # Show current person identity
```

## Shard Discovery

```bash
flint shard list                      # List installed + dev shards
flint shard info <sh>                 # Detailed shard info
flint shard status <sh>               # Status + pending migrations
flint shard check                     # Check shard system integrity
```

## Shard Manifests (loading shards)

`start` / `hstart` assemble a dynamic manifest from each shard's files (init, skills, workflows, templates, knowledge — read from each file's `description` frontmatter). Run the variant that matches your mode.

```bash
flint shard start <name>              # Installed, interactive (loads init-<sh>.md)
flint shard start-dev <name>          # Dev, interactive (loads dev-init-<sh>.md)
flint shard hstart <name>             # Installed, headless (loads hinit-<sh>.md, prefers hwkfl-*)
flint shard hstart-dev <name>         # Dev, headless
```

If a shard declares `setup:` and isn't set up yet, the manifest output appends a `SETUP REQUIRED` banner pointing at `setup-<sh>.md`.

## Shard Install / Update

```bash
flint shard install <source>          # Install from owner/repo or path
flint shard install --all-dev         # (Re)install all dev shards into their Shards/<Name>/ copies
flint shard reinstall <name>          # Re-run install entries (after new install files are added)
flint shard update                    # Update installed shards to latest published versions
flint shard uninstall <sh>            # Remove shard and clean files
flint shard heal <name>               # Auto-repair shard issues (folders, manifest, naming, deps)
```

## Shard Versioning

```bash
flint shard versions <sh>             # List remote versions
flint shard pin <sh> <version>        # Lock shard to a specific version
flint shard unpin <sh>                # Remove version lock
```

## Shard Migrations

```bash
flint shard migrate list <sh>         # List pending migrations for an installed shard
flint shard migrate run <sh>          # Run the next pending migration
```

## Shard Scripts

Shards can ship Node.js scripts under `scripts/`, auto-discovered and invoked via the shard's shorthand:

```bash
flint shard scripts <sh>              # List executable scripts for a shard
flint shard <sh> <script> [args...]   # Run a declared script
```

> Authoring commands (`create`, `clone`, `dev`, `rename`, `push`, `pull`, `release`, `publish`, `unpublish`, `published`) are documented by the Knap shard — load `Shards/Knap/init-knap.md` when authoring shards.

## Sync

```bash
flint sync                            # Sync all shards and mods from flint.toml
```

## Workspace

```bash
flint workspace                       # Manage workspace references (codebases, URLs)
flint subflint                        # Manage subflints (nested Flints)
flint connection                      # Manage connections to other Flints
```

## Send / Inbox

```bash
flint send <target> <title> <instructions> <files...>  # Send files to another Flint's inbox as a bundle
flint send "My Other Flint" "Auth Migration" "Move tasks to Tasks/" file1.md file2.md
```

Files land in the target Flint's `Inbox/(Bundle) Title/` directory. Use [[dev-sk-f-inbox_process]] to process incoming bundles.

## Other Useful Commands

```bash
flint repair                          # Repair flint to match current specs
flint open                            # Open flint in configured applications
flint export                          # Manage exports
flint import                          # Manage imports from other flints
```
