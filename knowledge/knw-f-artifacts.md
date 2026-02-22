# Knowledge: Artifact Conventions

Complete reference for artifact frontmatter, tags, status management, naming conventions, and session tracking in Flint workspaces.

## What Are Artifacts

Artifacts are the documents that agents create and manage inside a Flint. Every artifact has:
- A **filename** following a naming convention
- **YAML frontmatter** with metadata
- **Markdown body** with structured content
- A **template** that defines its structure

## Naming Conventions

### Typed Artifacts

Most artifacts use the typed naming pattern:

```
(Type) NNN Name.md
```

| Component | Format | Example |
|-----------|--------|---------|
| Type | Title Case in parens | `(Task)`, `(Plan)`, `(Notepad)` |
| Number | 3-digit zero-padded | `001`, `042`, `103` |
| Name | Descriptive title | `Implement auth`, `Q1 Roadmap` |

**Getting the next number:**
```bash
flint helper type newnumber Task     # → 076
flint helper type newnumber Plan     # → 004
flint helper type newnumber Notepad  # → 028
```

Use `--raw` for an unpadded number.

**Special case — Increments:**
```
(Increment) X.N.I - Name.md
(Increment) 3.16.0 - Shard Registry.md
(Increment) 3.A.0 - Adhoc.md
```

Increments use semantic version numbers instead of sequential IDs.

### Concept Notes (Untyped)

Notes without a type prefix are concept notes:

```
Concept Title.md
```

These live in `Mesh/Notes/` and their title IS the concept — a complete phrase that stands alone.

Good: `Context windows limit agent memory.md`
Bad: `Notes from meeting.md`

### Dashboards and System Files

```
(Dashboard) Backlog.md              # DataviewJS live view
(System) Flint Init.md              # System-managed file
```

These use the `(Type)` pattern but without numbers.

## Frontmatter Schema

Every typed artifact has YAML frontmatter between `---` delimiters:

```yaml
---
id: a1b2c3d4-e5f6-7890-abcd-ef1234567890
tags:
  - "#task"
  - "#proj/task"
status: in-progress
increment: "[[(Increment) 3.16.0 - Shard Registry]]"
due: 2026-03-01
completed:
priority: high
claude-sessions:
  - session-id-1
  - session-id-2
template: tmp-proj-task
---
```

### Required Fields

| Field | Type | Purpose |
|-------|------|---------|
| `id` | UUID v4 | Permanent identifier, never changes |
| `tags` | string[] | Classification and discovery |
| `template` | string | Filename stem of the governing template |

### Common Optional Fields

| Field | Type | Purpose |
|-------|------|---------|
| `status` | enum string | Lifecycle state (defined per template) |
| `increment` | wikilink string | Parent increment: `"[[(Increment) X.N.I - Name]]"` |
| `due` | ISO 8601 date | Deadline |
| `completed` | ISO 8601 date | Completion date |
| `priority` | `low\|medium\|high` | Priority level |
| `[agent]-sessions` | string[] | Session IDs of agents that edited this file |
| `artifacts-created` | wikilink[] | Used by notepads to track outputs |

## Tags

Tags use the `#prefix/type` convention and are always quoted in YAML (because `#` starts a comment):

```yaml
tags:
  - "#task"
  - "#proj/task"
```

### Tag Patterns

| Pattern | Meaning | Examples |
|---------|---------|---------|
| `#type` | Artifact type | `#task`, `#plan`, `#notepad`, `#note` |
| `#shard/artifact` | Shard-specific type | `#proj/task`, `#ntpd/notepad`, `#inc/stream` |
| `#shard/state` | Shard-specific state | `#inc/active`, `#inc/complete`, `#ld/living` |
| `#system` | System-managed file | `#system`, `#core/init` |

### Shard Tags

Each shard defines its own tag namespace using its shorthand:
- Projects (`proj`): `#proj/task`
- Increments (`inc`): `#inc/stream`, `#inc/checkpoint`, `#inc/active`, `#inc/complete`
- Notepad (`ntpd`): `#ntpd/notepad`
- Living Documents (`ld`): `#ld/living`, `#ld/dead`
- Plans (`plan`): `#plan/plan`

## Status Management

Each artifact type defines its own status lifecycle. Common patterns:

### Linear Lifecycle
```
draft → todo → in-progress → review → done
```

### With Branching
```
draft → todo → in-progress → review → done
                    ↓            ↑
                 blocked ────────┘

Any status → deprecated (terminal)
```

### Binary Lifecycle
```
active → archived
```

### Status Rules

1. **Templates define available statuses** — check the template for valid options
2. **Init files document transitions** — the shard's init explains when to transition
3. **Skills/workflows handle transitions** — don't change status ad-hoc; use the appropriate skill
4. **Terminal states** — `done`, `archived`, `deprecated`, `dead` are typically terminal

## Session Tracking

Agents track their involvement by appending session IDs to the frontmatter.

### How It Works

1. When editing a file with frontmatter, check for a `[agent]-sessions` field
2. If it exists, append your session ID to the list
3. If it doesn't exist, create it with your session ID

```yaml
# Before
claude-sessions:
  - previous-session-id

# After
claude-sessions:
  - previous-session-id
  - your-current-session-id
```

### Agent Types

| Agent | Field |
|-------|-------|
| Claude (terminal) | `claude-sessions` |
| Codex | `codex-sessions` |
| Custom | `[name]-sessions` |

### When to Track

- When you **create** a new artifact
- When you **substantively edit** an artifact (status changes, content additions)
- Do NOT track when you only read a file

## Wikilinks

Flint uses Obsidian-style wikilinks for cross-references:

```markdown
[[Document Title]]                   # Link by title
[[(Task) 076 Implement auth]]        # Link to typed artifact
[[(Increment) 3.16.0 - Shard Registry]] # Link to increment
```

In frontmatter, wikilinks are quoted:

```yaml
increment: "[[(Increment) 3.16.0 - Shard Registry]]"
artifacts-created:
  - "[[(Task) 104 Implement shard pull]]"
```

## Archiving

When artifacts reach terminal states, they typically move to `Mesh/Archive/`:

- `Mesh/Archive/Tasks/` — Completed tasks
- `Mesh/Archive/Plans/` — Spent plans
- etc.

The archive mirrors the `Types/` structure. The artifact's status is updated before moving.
