# Knowledge: Flint Workspace Structure

Deep reference for how a Flint workspace is organized, what each part contains, and how agents should navigate it.

## Workspace Anatomy

A Flint workspace is a directory that contains a `.flint/` configuration folder, a `Mesh/` content folder, and a `Shards/` capabilities folder. Everything an agent interacts with is organized under these top-level directories.

```
Flint Root/
├── .flint/                         # Internal — managed by CLI
│   ├── flint.config.yaml           # Runtime config (version, feature flags)
│   └── shards/                     # Shard installation metadata
│       └── <shorthand>/
│           └── installed.json      # Install state tracking
├── Mesh/                           # All workspace content
│   ├── (System) Flint Init.md      # Workspace overview — READ FIRST
│   ├── (Dashboard) *.md            # Live dashboards (DataviewJS)
│   ├── Types/                      # Typed artifacts organized by category
│   │   ├── Tasks/                  # (Task) NNN Name.md
│   │   ├── Plans/                  # (Plan) NNN Name.md
│   │   ├── Notepads/               # (Notepad) NNN Name.md
│   │   ├── Increments/             # (Increment) X.N.I - Name.md
│   │   ├── Workspaces/             # (Workspace) NNN Name.md
│   │   └── Reports/                # (Report) NNN Name.md
│   ├── Notes/                      # Concept notes (untyped, evergreen)
│   ├── Archive/                    # Completed/archived artifacts
│   │   ├── Tasks/
│   │   ├── Plans/
│   │   └── ...
│   └── [Other folders]             # Shard-specific or custom folders
├── Shards/                         # Installed capabilities
│   ├── Core/                       # Always present
│   ├── Projects/                   # Task management
│   ├── Increments/                 # Version tracking
│   ├── (Dev) My Shard/             # Dev shards (local authoring)
│   ├── (System) Obsidian Templates/ # Human-facing templates (otmp- prefix)
│   ├── (System) Repos/             # Cloned Git repos from shards (gitignored)
│   └── ...
├── Subflints/                      # Nested Flint workspaces
│   └── (Flint) Sub Project/
├── flint.toml                      # Flint configuration file
└── CLAUDE.md                       # Agent-specific instructions (optional)
```

## The Mesh

The `Mesh/` directory is where all workspace content lives. Agents read and write to this directory. It contains:

### System Files
- `(System) Flint Init.md` — The workspace overview. Every session should read this first. It contains what the Flint is about, how to navigate it, what shards are installed, workspace references, and development commands.
- `(System) Plugin Status.md` — Shard installation status (legacy, may not exist in all Flints).

### Dashboards
Files prefixed with `(Dashboard)` are live views powered by DataviewJS in Obsidian. They aggregate and display artifacts by status, tag, or other criteria. Agents don't usually edit dashboards — they update the artifacts that dashboards query.

Common dashboards:
- `(Dashboard) Backlog.md` — Tasks by status
- `(Dashboard) Increments.md` — Version history
- `(Dashboard) Plans.md` — Strategic plans
- `(Dashboard) Notepads.md` — Brainstorming sessions
- `(Dashboard) Workspaces.md` — Active development contexts

### Types
The `Types/` folder organizes typed artifacts into subfolders by their type prefix. When creating a new typed artifact, check for an existing subfolder:
- `Types/Tasks/` for `(Task)` files
- `Types/Plans/` for `(Plan)` files
- `Types/Notepads/` for `(Notepad)` files
- etc.

If no subfolder exists for a type, create the artifact directly in `Mesh/`.

### Notes
`Notes/` contains concept notes — untyped, evergreen notes that capture atomic ideas. These are the fundamental knowledge building blocks. See [[tmp-f-note]] for the template.

### Archive
`Archive/` stores completed or deprecated artifacts. When artifacts reach terminal states (done, archived, deprecated), they move here. Archive mirrors the Types structure:
- `Archive/Tasks/` for completed tasks
- `Archive/Plans/` for spent plans

## flint.toml

The workspace configuration file. Key sections:

```toml
[flint]
name = "My Flint"                    # Display name
flintVersion = "1.2.0"              # Flint framework version

[shards]
# Installed shards and their sources
core = { source = "builtin" }
projects = { source = "builtin" }
my-shard = { source = "github:owner/repo" }
dev-shard = { source = "path:./Shards/(Dev) My Shard", dev = true }

[workspace]
# External references (codebases, APIs, etc.)
references = [
  { name = "monorepo", type = "codebase", path = "/path/to/repo" },
  { name = "api-docs", type = "url", url = "https://docs.example.com" },
]
```

Workspace references tell agents about external codebases and resources that this Flint relates to. Agents should use these references when exploring beyond the Flint itself.

## CLAUDE.md

If present at the Flint root, `CLAUDE.md` contains agent-specific instructions that apply to every session. This file is loaded automatically by Claude Code and may contain:
- Workspace-specific conventions
- Which shards to always load
- How to handle sessions in this particular Flint
- References to key files

## Subflints

Nested Flint workspaces that live under `Subflints/`. Each subflint is a complete Flint with its own `.flint/`, `Mesh/`, and `Shards/`. Subflints are independent but can reference the parent Flint's resources.

## Navigation Strategy

1. **Start with Init** — Read `(System) Flint Init.md` for overview
2. **Search by name** — Use glob patterns to find files by their `(Type)` prefix
3. **Search by tag** — Use `mesh query --tag X` for tag-based discovery
4. **Search by content** — Use grep for content-based search
5. **Check dashboards** — Dashboards aggregate current state
6. **Avoid directory walking** — Don't `ls` through deep trees; search directly
