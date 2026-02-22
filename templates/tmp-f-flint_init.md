# Filename: Mesh/(System) Flint Init.md

/* This is the workspace overview file. It's the first thing every agent reads when starting a session.
   It should provide a complete, concise overview of what this Flint is about and how to navigate it.

   This file is installed with `once: true` — it gets created once, then maintained by agents using
   sk-f-init_update and sk-f-init_full_update.
*/

```markdown
---
id: [generate-uuid4]
tags:
  - "#system"
  - "#core/init"
---

/* Flint-specific instructions go here — anything unique to this workspace that agents need to know. */

# [Flint Name]

[1-2 sentence description of what this Flint is about and its primary purpose.]

## Monorepo Overview

/* If the Flint relates to a codebase, describe the codebase structure here. */

[Description of the codebase structure, key packages, and how they relate.]

### Core Packages

| Package | Purpose |
|---------|---------|
| [package-name] | [what it does] |
| (continue) |

### Applications

| App | Purpose |
|-----|---------|
| [app-name] | [what it does] |
| (continue) |

## Current State

### Active Increment

**[X.N.I - Name]** — [Brief description of what this increment is working toward.]

### Active Workspaces

/* List only active workspaces, not completed ones */

1. **(Workspace NNN) [Name]** — [Status/phase]
2. (continue)

### Recent Milestones

- [Milestone description]
- (continue)

## Navigation

### Dashboards

- `(Dashboard) [Name]` — [Purpose]
- (continue)

### Key Folders

- `Mesh/Types/Tasks/` — [Active range]
- `Mesh/Types/Plans/` — [Plan status]
- `Mesh/Types/Increments/` — [Version range]
- (continue)

### Subflints

/* If subflints exist */

- `(Flint) [Name]` — [Purpose]
- (continue)

## Shards

| Shard | Shorthand | Purpose |
|-------|-----------|---------|
| [Name] | `[sh]` | [What it does] |
| (continue) |

**Load shards on-demand:** `[[init-[shorthand]]]`

## Workspace References

| Reference | Path | Type |
|-----------|------|------|
| [name] | [path or URL] | [codebase\|url\|api] |
| (continue) |

## Development Commands

/* If the Flint relates to a codebase */

\`\`\`bash
[command]    # [what it does]
(continue)
\`\`\`
```
