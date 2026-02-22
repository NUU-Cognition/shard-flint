# Flint Agent Environment

You are a terminal-based AI agent operating inside a Flint workspace. This shard defines your foundational capabilities — how to navigate the workspace, use shards, read templates, manage sessions, and create artifacts.

## Your Environment

- **Working directory**: Flint root (parent of `Mesh/`, `Shards/`, etc.)
- **Capabilities**: Read/write files, run shell commands, search codebase
- **Context window**: Limited — load files on demand, never assume prior knowledge
- **Session**: Stateless — each conversation starts fresh

## First Action

Always read `Mesh/(System) Flint Init.md` first. It contains:
- What this Flint is about
- How to navigate it
- What shards are installed
- Flint-specific instructions and workspace references

## Workspace Structure

```
Flint Root/
├── .flint/                    # Internal config (don't edit directly)
├── Mesh/                      # All workspace content
│   ├── (System) Flint Init.md # Workspace overview (read first)
│   ├── (Dashboard) *.md       # DataviewJS dashboards
│   ├── Types/                 # Typed artifacts by category
│   │   ├── Tasks/
│   │   ├── Plans/
│   │   ├── Notepads/
│   │   ├── Increments/
│   │   └── ...
│   ├── Notes/                 # Concept notes (untyped)
│   ├── Archive/               # Completed/archived artifacts
│   └── ...                    # Other workspace content
├── Shards/                    # Installed shards (capabilities)
│   ├── Core/
│   ├── Projects/
│   ├── (Dev) My Shard/        # Dev shards (local authoring)
│   └── ...
├── Subflints/                 # Nested Flint workspaces
└── flint.toml                 # Flint configuration
```

For deep knowledge about workspace structure, see [[knw-f-workspace]].

## Navigation

Search for files directly rather than browsing directory trees. Use:
- `mesh query --tag X` — Find notes by tag
- File name patterns — Most artifacts follow `(Type) NNN Name.md`
- Glob search — Find files by pattern

## Shard System

Shards extend your capabilities with context, skills, templates, workflows, and knowledge. Each shard is a self-contained unit focused on one domain.

**Loading shards**: Shards are loaded **on demand**, not all at once. Only init shards relevant to the current task.

1. Load context: `@Shards/[Name]/init-[shorthand].md`
2. Use skills: `@Shards/[Name]/skills/sk-[shorthand]-[name].md`
3. Use workflows: `@Shards/[Name]/workflows/wkfl-[shorthand]-[name].md`
4. Use templates: `@Shards/[Name]/templates/tmp-[shorthand]-[name].md`
5. Read knowledge: `@Shards/[Name]/knowledge/knw-[shorthand]-[name].md`

**Discovering shards**: List the folders in `Shards/` or run `flint shard list`.

**Important**: Always read a shard's init file before using any of its skills, workflows, or templates.

### Shard File Types

| Pattern | Purpose | When to Use |
|---------|---------|-------------|
| `shard.yaml` | Shard manifest | Defines name, version, dependencies, install behavior |
| `init-[sh].md` | Context file | Load when you need that shard's capabilities |
| `sk-[sh]-[name].md` | Skill | Atomic, single-purpose tasks (no human checkpoints) |
| `wkfl-[sh]-[name].md` | Workflow | Multi-stage tasks with human review points |
| `tmp-[sh]-[name].md` | Template | Structural guide for creating artifacts |
| `knw-[sh]-[name].md` | Knowledge | Deep reference material on a topic |
| `ast-[sh]-[name].[ext]` | Asset | Non-markdown files (images, scripts, data) |

## Files and Artifacts

### Typed Artifacts

Files inside Flint are typed using a prefix: `(Type) NNN Name.md`

```
(Task) 076 Implement authentication.md
(Plan) 003 Shard Registry.md
(Notepad) 015 Planning session.md
(Increment) 3.2.1 - Feature development.md
```

**ID generation**: Use `flint helper type newnumber <Type>` to get the next available number (zero-padded, 3 digits).

```bash
TASKNUM=$(flint helper type newnumber Task)
# Creates: (Task) $TASKNUM My Task.md
```

**File location**: When creating typed files, check whether a folder for that type exists (e.g., `Mesh/Types/Tasks/`). If so, create the file there.

### Concept Notes (Untyped)

Notes without a `(Type)` prefix are **concept notes** — atomic, evergreen notes that capture a single idea. They live in `Mesh/Notes/` and follow [[tmp-f-note]].

Key principles:
- **Atomic** — one concept per note; two ideas = two notes, linked
- **Concept-oriented** — title IS the concept, a complete phrase
- **Densely linked** — every note should `[[link]]` to related concepts
- **Associative** — no forced hierarchy; structure emerges from links
- **Evolving** — revisit, refine, extend over time

Use [[sk-f-create_note]] to create concept notes. Always check `Mesh/Notes/` first to avoid duplicates.

## Template Syntax

When editing an artifact with a `template` field in its frontmatter, find and read the corresponding template file. Templates use a special syntax for agent generation.

For the complete template syntax reference, see [[knw-f-templates]]. Quick summary:

| Pattern | Meaning | Output |
|---------|---------|--------|
| `[instruction]` | Generate text per instruction | Plain text (no brackets) |
| `[opt1\|opt2\|...]` | Pick one option | Selected option |
| `[opt1 (desc)\|opt2 (desc)]` | Options with descriptions | Selected option (no desc) |
| `[[instruction]]` | Generate an Obsidian wikilink | `[[Document Title]]` |
| `(continue)` | Repeat preceding pattern if needed | More items or stop |
| `/* comment */` | Agent-only instruction | Not included in output |

## Frontmatter Conventions

All typed artifacts include standard YAML frontmatter. For the complete reference, see [[knw-f-artifacts]]. Key fields:

```yaml
id: [uuid]                           # Permanent identifier
tags:                                 # Classification
  - "#[type]"                        # e.g., #task, #notepad
  - "#[shard]/[artifact]"            # e.g., #proj/task
status: [state]                       # Lifecycle state
increment: [[parent increment]]       # Parent version (if inc shard installed)
template: tmp-[sh]-[name]            # Template this artifact follows
[agent]-sessions:                     # Session tracking
  - [session-id]
```

### Session Tracking

When editing files with frontmatter, append your session ID to the `[agent]-sessions` field:
- If you are a Claude session: `claude-sessions: [your-session-id]`
- If you are a Codex session: `codex-sessions: [your-session-id]`
- If the field doesn't exist, create it. If it exists, append your ID.

## Operating Principles

1. **Read before writing** — understand existing patterns before making changes
2. **Follow naming conventions** — use existing patterns for file names, tags, and structure
3. **Tag documents appropriately** — every artifact gets proper tags
4. **Write outputs to Mesh/** — all workspace content lives under Mesh/
5. **Use relative paths** — from flint root
6. **Load shards on demand** — don't load what you don't need
7. **Search, don't browse** — find files by name/tag, not by walking directories

## Available Commands

```bash
# Flint CLI
flint shard list                      # List installed shards
flint shard <shorthand>               # Show shard info
flint helper type newnumber <Type>    # Get next artifact number

# Mesh CLI
mesh query --tag <tag>                # Find notes by tag
mesh compile <file>                   # Compile a mesh document

# Standard tools
git, npm, pnpm, etc.                  # Standard development tools
```

## Skills

| Skill | File | Purpose |
|-------|------|---------|
| Create Note | `sk-f-create_note.md` | Create an evergreen concept note |
| Init Update | `sk-f-init_update.md` | Update the Flint Init from current session |
| Init Full Update | `sk-f-init_full_update.md` | Research entire Flint and rewrite Init |
| Upgrade | `sk-f-upgrade.md` | Upgrade Flint to latest version |

## Templates

| Template | File | Purpose |
|----------|------|---------|
| Concept Note | `tmp-f-note.md` | Atomic evergreen concept note |
| Flint Init | `tmp-f-flint_init.md` | `(System) Flint Init.md` structure |

## Knowledge

| Knowledge | File | Topic |
|-----------|------|-------|
| Workspace Structure | `knw-f-workspace.md` | Flint workspace anatomy and conventions |
| Template Syntax | `knw-f-templates.md` | Complete guide to reading and generating from templates |
| Artifact Conventions | `knw-f-artifacts.md` | Frontmatter, tags, status, naming, session tracking |
