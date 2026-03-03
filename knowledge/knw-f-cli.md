# Knowledge: CLI Reference

Quick reference for Flint CLI commands relevant to agent workflows.

## Artifact Helpers

```bash
flint helper type newnumber <Type>    # Next number for a typed artifact (zero-padded, 3 digits)
flint helper type newnumber Task      # → 119
flint helper type newnumber Notepad   # → 034
```

## Shard Management

```bash
flint shard list                      # List installed shards
flint shard <name>                    # Show shard overview
flint shard info <name>               # Detailed shard info
flint shard reinstall <name>          # Re-run install entries (useful after adding new install files)
flint shard pull [name]               # Sync shard repos and install entries
flint shard check                     # Check shard system integrity
flint shard heal <name>               # Fix a shard — folders, manifest, naming, dependencies
```

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

## Other Useful Commands

```bash
flint repair                          # Repair flint to match current specs
flint open                            # Open flint in configured applications
flint export                          # Manage exports
flint import                          # Manage imports from other flints
```
