---
description: "Frontmatter, tags, naming, notes, and session tracking conventions"
---

# Knowledge: Artifact Conventions

Complete reference for artifact frontmatter, tags, naming, notes, and session tracking in a Flint.

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

**Creating a numbered artifact:**
```bash
flint helper artifact create Task --title "Implement auth"   # → (Task) 076 Implement auth.md
flint helper artifact create Plan --title "Q1 Roadmap"       # → (Plan) 004 Q1 Roadmap.md
```

`flint helper artifact create` reserves the number and writes the file in one step, so two agents never get the same number. Then fill the file. `flint helper type newnumber <Type>` only reads the next number (a preview). Do not use that number to name a file yourself.

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
orb:
  - "@v1/nuucognition/session/154d8abd-6089-4b1d-9969-6cb51dfe926b#flint/nuu-flint"
  - "@v1/nuucognition/session/60be4985-9cd6-42f1-8245-e56c1ad68324#flint/nuu-flint"
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
| `authors`           | string[]            | Person wikilinks from the Name of this machine (`flint whoami`) |
| `orb`               | string[]            | The v1 addresses of the agent sessions that edited this file (`flint address me`) |
| `orbh-sessions`     | string[]            | The old session field (`"[[<uuid>]]"` items). Old notes keep it; no new item goes into it |

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
- Flint (`f`): `#note` (see the Invironments shard for the `#ie/` structure namespace)
- Projects (`proj`): `#proj/task`
- Increments (`inc`): `#inc/stream`, `#inc/checkpoint`, `#inc/active`, `#inc/complete`
- Notepad (`ntpd`): `#ntpd/notepad`
- Plans (`plan`): `#plan/plan`

## Notes

Notes are the base unit of free-form content in a Flint — one editable model per file, tagged `#note`. **Note structure and placement are owned by the Invironments shard (`ie`)**, a dependency of Flint: every note carries exactly one `#ie/sections/<name>` tag saying which mesh section it lives in, and a note created with no destination goes to the **New** section (`Mesh/Main/`). Use `flint shard ie note` to create notes and [[tmp-ie-note-v0.1]] for their structure. There are no note subtypes — one flat `#note`, with structure added only when it pays off.

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

Agents stamp their sessions in the frontmatter field `orb:`. An item of `orb:` is the v1 address of one session. All agent types use this one field.

### How It Works

1. Run `flint address me`. Its `by` line is the address of your session (`flint address me --json` gives `{ by, as, org }`).
2. When the file has no `orb:` field, create it with your address.
3. When the file has `orb:`, append your address to the list.
4. Add nothing only when `orb:` or `orbh-sessions` has an item that is exactly one of these forms (with no quotes, trimmed): your `by` address, the short form `@v1/<org>/session/<uuid>`, `[[<uuid>]]`, or your bare `<uuid>` (the UUID in lower case or capitals). Each other item is no stamp of your session, so append your address.
5. Do not parse an address to decide (R27 of NUU Orbh Mission 027). An item can look like your session and still be another thing: a wisp with the same UUID, an address with a bad place, or text around an address. A rare duplicate of another valid form is accepted: each reader matches two items by kind and key, also when their org or place is different.

```yaml
# Before
orb:
  - "@v1/nuucognition/session/154d8abd-6089-4b1d-9969-6cb51dfe926b#flint/nuu-flint"

# After
orb:
  - "@v1/nuucognition/session/154d8abd-6089-4b1d-9969-6cb51dfe926b#flint/nuu-flint"
  - "@v1/nuucognition/session/<your-session-uuid>#flint/nuu-flint"
```

### The Address

- The form is `@v1/<org>/session/<uuid>#flint/<slug>`. The place `#flint/<slug>` names the Flint of the session (a Computer Flint has `#flint.computer/<slug>`).
- The org is the org of the Flint. When the Flint has no org and the person is logged in, it is the org of the person (`flint org set <org>` makes it permanent). Else the org is empty: `@v1//session/<uuid>#flint/<slug>`.
- Quote each item. YAML does not permit a plain value that starts with `@`.
- An item is an address, not a wikilink. Obsidian shows no link from the note to the session.
- Stamp only `by` (the session). Do not stamp `as` (a wisp or an agent): Orbh finds the wisp of a session from the session.
- With a CLI that has no `flint address me` (a release before the v1 addresses), write the short form with no place: `"@v1/<org>/session/<ORBH_SESSION_ID>"`, with the org of `flint.toml` (`[flint] org`), else the empty org.

### The Old Field: `orbh-sessions`

Notes from before the v1 addresses have the field `orbh-sessions`, with `"[[<uuid>]]"` items (some have bare UUIDs). These notes are not migrated.

- Never remove `orbh-sessions` from a note, and never change its items.
- Add no new item to `orbh-sessions`. A new stamp goes into `orb:`, also in a note that has `orbh-sessions`.
- A reader of the sessions of a note reads both fields.

### One Field for All Runtimes

All agent runtimes (Claude Code, Codex, etc.) write to the same `orb:` field. There are no per-runtime session fields. Orbh sessions map to native runtime sessions via the runs model in the session JSON.

### When to Track

- When you **create** a new artifact
- When you **substantively edit** an artifact (status changes, content additions)
- Do NOT track when you only read a file
- Do NOT track on session init — reading files at startup is not a substantive edit

## Authorship

Artifacts track who created or substantively edited them via the `authors` frontmatter field.

### How It Works

1. Resolve the Name of the person ("your Name" in the screens of the CLI) — run `flint whoami`, or read the `name` key of the NUU config (`config.toml` in the NUU home, `~/.nuucognition` or `$NUU_HOME`) — and form the person as `@<Name>` (e.g. `@Nathan Luo`)
2. When creating an artifact, add the person as a wikilink to `authors`
3. When substantively editing an artifact, add the person if not already listed
4. If no Name is set, omit the `authors` field entirely. The operator sets the identity of the machine with `flint setup`, and changes the Name only with `flint config name "<Name>"`.

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

The `@` prefix is the naming convention. These files can be empty — the filename IS the identity, and the marker is auto-created on first use. The Name is the same for every Flint on this machine: it lives in the NUU config (`config.toml` in the NUU home). The `@Person` is `@<Name>`.

### Rules

- `authors` is always a list (even with one person)
- The person is the author, not the agent — agents act *on behalf of* the person
- Do NOT add authors when only reading a file
- Do NOT confuse authors with `orb:` (or the old `orbh-sessions`) — sessions track agent involvement, authors track human attribution

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
