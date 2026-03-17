# Knowledge: Artifact Conventions

Complete reference for artifact frontmatter, tags, naming, notes, and session tracking in Flint workspaces.

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

**Special case — Increments:**
```
(Increment) X.N.I - Name.md
(Increment) 3.16.0 - Shard Registry.md
(Increment) 3.A.0 - Adhoc.md
```

Special cases will describe how they increment in numbers in their unique init files. 

Typed files will describe where they should go in the mesh in their templates. 
### Notes (Untyped)

Notes are the bare and most default type inside of a flint. Notes do not use a `(Type)` prefix. 

```
Title of the note.md
```

Notes are the base unit of free-form content. Their title should be descriptive and stand alone. Notes are differentiated by tags, not by folders or naming conventions. See the **Notes** section below.

### Dashboards and System Files

```
(Dashboard) Backlog.md              # DataviewJS live view
(System) Flint Init.md              # System-managed file
```

These use the `(Type)` pattern but without numbers.

## Frontmatter Schema

Every artifact has YAML frontmatter between `---` delimiters. For example: 

```yaml
---
id: a1b2c3d4-e5f6-7890-abcd-ef1234567890
tags:
  - "#proj/task"
status: in-progress
increment: "[[(Increment) 3.16.0 - Shard Registry]]"
due: 2026-03-01
completed:
priority: high
claude-sessions:
  - "[[154d8abd-6089-4b1d-9969-6cb51dfe926b]]"
  - "[[60be4985-9cd6-42f1-8245-e56c1ad68324]]"
template: "[[tmp-proj-task-v0.1]]"
authors:
  - "[[@Nathan Luo]]"
---
```

### Required Fields

| Field | Type | Purpose |
|-------|------|---------|
| `id` | UUID v4 | Permanent identifier, never changes |
| `tags` | string[] | Classification and discovery |

### Common Optional Fields

| Field               | Type                | Purpose                                     |
| ------------------- | ------------------- | ------------------------------------------- |
| `template`          | string              | Filename stem of the governing template     |
| `status`            | enum string         | Lifecycle state (defined per template)      |
| `due`               | ISO 8601 date       | Deadline                                    |
| `completed`         | ISO 8601 date       | Completion date                             |
| `priority`          | `low\|medium\|high` | Priority level                              |
| `authors`           | string[]            | Person wikilinks from `.flint/identity.json` |
| `[agent]-sessions`  | string[]            | Session IDs of agents that edited this file |

## Tags

Tags use the `#prefix/type` convention and are always quoted in YAML (because `#` starts a comment):

```yaml
tags:
  - "#task"
  - "#proj/task"
```

### Tag Patterns

| Pattern           | Meaning              | Examples                                     |
| ----------------- | -------------------- | -------------------------------------------- |
| `#shard/artifact` | Shard-specific type  | `#proj/task`, `#ntpd/notepad`, `#inc/stream` |
| `#note/subtype`   | Note subtype         | `#note/concept`, `#note/record`              |
| `#shard/state`    | Shard-specific state | `#inc/active`, `#inc/complete`               |

### Shard Tags

Each shard defines its own tag namespace using its shorthand:
- Flint (`f`): `#note`, `#note/concept`, `#note/record`
- Projects (`proj`): `#proj/task`
- Increments (`inc`): `#inc/stream`, `#inc/checkpoint`, `#inc/active`, `#inc/complete`
- Notepad (`ntpd`): `#ntpd/notepad`
- Plans (`plan`): `#plan/plan`

## Notes

Notes are the base unit of free-form content in a Flint. They live in `Mesh/Notes/` and are differentiated purely by tags.

### Base Note

Any file in `Mesh/Notes/` with an `id` in frontmatter. A note is just a block of text — no required structure beyond having an ID and tags. The default Obsidian template (`otmp-f-note`) creates bare notes with just a UUID.

### Concept Notes

A concept note represents an atomic idea — a single concept that can be linked to other concepts. Tagged `#note/concept`.

| Property | Rule |
|----------|------|
| Title | IS the concept — a complete phrase that stands alone |
| Scope | One concept per note; two ideas = two notes, linked |
| Links | Densely linked to related concepts via `[[wikilinks]]` |
| Lifecycle | Evergreen — revisit, refine, extend over time |

Use [[tmp-f-concept]] when creating concept notes.

Good titles: `Context windows limit agent memory`, `Shards extend agent capabilities`
Bad titles: `Notes from meeting`, `Misc thoughts`

### Record Notes

A record note captures a fact, event, or observation. Tagged `#note/record`.

| Property | Rule |
|----------|------|
| Title | Describes what was recorded |
| Scope | One fact/event per note |
| Links | Links to related concepts or artifacts |
| Lifecycle | Generally static once created |

Use [[tmp-f-record]] when creating record notes.

Good titles: `API rate limit is 1000 requests per minute`, `2026-03-03 Shard template install verified`

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
## Session Tracking

Agents track their involvement by appending session IDs to the frontmatter.

### How It Works

1. When editing a file with frontmatter, check for a `[agent]-sessions` field
2. If it exists, append your session ID to the list
3. If it doesn't exist, skip this step

```yaml
# Before
claude-sessions:
  - "[[previous-session-id]]"

# After
claude-sessions:
  - "[[previous-session-id]]"
  - "[[your-current-session-id]]"
```

### Agent Types

| Agent             | Field             |
| ----------------- | ----------------- |
| Claude (terminal) | `claude-sessions` |
| Codex             | `codex-sessions`  |

### When to Track

- When you **create** a new artifact
- When you **substantively edit** an artifact (status changes, content additions)
- Do NOT track when you only read a file
- Do NOT track on session init — reading files at startup is not a substantive edit

## Authorship

Artifacts track who created or substantively edited them via the `authors` frontmatter field.

### How It Works

1. Read `.flint/identity.json` to get the current person (e.g. `@Nathan Luo`)
2. When creating an artifact, add the person as a wikilink to `authors`
3. When substantively editing an artifact, add the person if not already listed
4. If no identity is set (file doesn't exist), omit the `authors` field entirely

```yaml
authors:
  - "[[@Nathan Luo]]"
  - "[[@Even Zhang]]"
```

### Person Files

People are represented by marker files in `Mesh/People/`:

```
Mesh/People/@Nathan Luo.md
Mesh/People/@Even Zhang.md
```

The `@` prefix is the naming convention. These files can be empty — the filename IS the identity. The `flint whoami "Name"` command creates the person file and writes `.flint/identity.json`.

### Rules

- `authors` is always a list (even with one person)
- The person is the author, not the agent — agents act *on behalf of* the person
- Do NOT add authors when only reading a file
- Do NOT confuse authors with `[agent]-sessions` — sessions track agent involvement, authors track human attribution

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

The archive mirrors the Types structure. The artifact's status is updated before moving.
