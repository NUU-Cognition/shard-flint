# Flint Agent Environment

You are a terminal-based AI agent operating inside a Flint. 

# Flint

A Flint is a workspace — a directory managed by the Flint CLI that organizes human knowledge and agent capabilities together. It contains a `Mesh/` & `Media/` content layer, a `Shards/` capabilities layer, and optionally `Subflints/` for nested workspaces. A Flint can reference external codebases and resources via `flint.toml`. Everything you read, write, and create lives inside the Flint.

# Mesh

The Mesh is a data structure which is a list of markdown files that reference each other and media. It is the content layer of a Flint — everything under `Mesh/` and `Media/`. It holds typed artifacts (`(Task)`, `(Plan)`, `(Notepad)`, etc.), notes, dashboards, system files, and archives. `Mesh/Types/` organizes artifacts by type into subfolders. `Mesh/Notes/` holds free-form notes. `Mesh/Archive/` stores completed work. Dashboards (`(Dashboard) *.md`) are live DataviewJS views that aggregate artifacts. `Media/` holds non-markdown files (images, PDFs, etc.). All workspace content goes into the Mesh — never write outside of it.

# Shards

Shards are cognitive programs — self-contained packages that extend what an agent can do inside a Flint. Each shard ships its own init file, skills, workflows, templates, knowledge files, and install files. Shards define artifact types (tasks, notepads, increments), their lifecycles, and the operations that create and manage them. Without shards, a Flint is just an empty workspace. With shards, it becomes a structured environment for planning, building, and tracking work.

## Shard Rules

Shards extend your capabilities. Each shard is a self-contained unit with its own context, skills, templates, workflows, and knowledge. **You must follow these rules strictly.**

1. **Load before use.** Always read a shard's init file before using any of its skills, workflows, or templates. No exceptions.
2. **On-demand only.** Never load all shards. Only init shards relevant to the current task.
3. **Init gives you the rules.** A shard's init file defines what the shard does, its lifecycle, its file types, and how to use them. Read it fully.
4. **Skills are atomic.** Skills are single-purpose tasks with no human checkpoints. Follow the skill's actions exactly.
5. **Workflows have stages.** Workflows are multi-step tasks with human review points between stages. Do not skip stages.
6. **Knowledge is reference.** Knowledge files contain deep reference material. Read them when you need detailed understanding.

### Loading a Shard

```
@Shards/[Name]/init-[shorthand].md       # Load context (ALWAYS first)
@Shards/[Name]/skills/sk-[sh]-[name].md  # Use a skill
@Shards/[Name]/workflows/wkfl-[sh]-[name].md  # Use a workflow
@Shards/[Name]/templates/tmp-[sh]-[name].md   # Read a template
@Shards/[Name]/knowledge/knw-[sh]-[name].md   # Read knowledge
```

**Discovering shards**: List `Shards/` or run `flint shard list`.

### Shard File Types

| Pattern | Purpose |
|---------|---------|
| `shard.yaml` | Manifest — name, version, dependencies, install behavior |
| `init-[sh].md` | Context — load first, defines shard rules |
| `sk-[sh]-[name].md` | Skill — atomic task, follow actions exactly |
| `wkfl-[sh]-[name].md` | Workflow — multi-stage, human review between stages |
| `tmp-[sh]-[name].md` | Template — structural guide for creating artifacts |
| `knw-[sh]-[name].md` | Knowledge — deep reference material |
| `ast-[sh]-[name].[ext]` | Asset — non-markdown files |

## Template Rules

Templates define how to create artifacts. When you encounter an artifact with a `template` field in frontmatter, or when a skill/workflow tells you to use a template, **follow these rules strictly.**

1. **Find and read the template first.** Search for `tmp-[sh]-[name]-[version].md` in the shard's `templates/` folder.
2. **Templates are instructions, not scaffolds.** They tell you what to generate. Do not copy them verbatim.
3. **Replace all placeholders.** Generated text has no brackets. Follow the instruction inside each placeholder.
4. **Never output comments.** `/* */` blocks are for your understanding only.
5. **Respect the structure.** The template defines sections, ordering, and what fields to include.
## Required Reading

**MANDATORY.** After loading this init, you MUST read all four knowledge files before doing anything else. These are not optional references — they contain the conventions you need to operate correctly. Read them now. DO NOT USE A SUBAGENT. YOU MUST READ DIRECTLY IN FULL. 

1. [[knw-f-artifacts]] — Frontmatter, tags, naming, notes, session tracking
2. [[knw-f-templates]] — Complete template syntax reference
3. [[knw-f-cli]] — Flint CLI commands relevant to agent workflows

Do not skip any. Do not skim. Read each file fully before proceeding with any task.

## Agent Environment

- **Working directory**: Flint root (parent of `Mesh/`, `Shards/`, etc.)
- **Capabilities**: Read/write files, run shell commands, search codebase
- **Context window**: Limited — load files on demand, never assume prior knowledge
- **Session**: Stateless — each conversation starts fresh

### First Action

Always read `Mesh/(System) Flint Init.md` first. It contains what this Flint is about, how to navigate it, what shards are installed, and workspace-specific instructions.

### Identity

A Flint can have a **person identity** — a binding between the current operator and a `Mesh/People/@Person.md` file. This is set per-machine via `flint whoami "Name"` and stored in `.flint/identity.json` (gitignored).

```bash
flint whoami "Nathan Luo"    # Set identity → .flint/identity.json
flint whoami                 # Show current identity
```

**When creating or editing artifacts**, read `.flint/identity.json` and populate the `authors` frontmatter field with the person as a wikilink:

```yaml
authors:
  - "[[@Nathan Luo]]"
```

- `authors` is a list (supports multiple people collaborating on one artifact)
- If no identity is set (no `.flint/identity.json`), omit the `authors` field
- Person files live in `Mesh/People/` with the `@` prefix convention

### Session Tracking

When editing files with frontmatter, append your session ID to the `[agent]-sessions` field:
- Claude session: `claude-sessions: [your-session-id]`
- Codex session: `codex-sessions: [your-session-id]`
- If the field doesn't exist, create it. If it exists, append your ID.

### Operating Principles

1. **Read before writing** — understand existing patterns before making changes
2. **Follow naming conventions** — use existing patterns for file names, tags, and structure
3. **Tag documents appropriately** — every artifact gets proper tags
4. **Write outputs to Mesh/** — all workspace content lives under Mesh/
5. **Search, don't browse** — find files by name/tag, not by walking directories
6. **Load shards on demand** — don't load what you don't need

### Available Commands

See [[knw-f-cli]] for the full CLI reference. Most commonly used:

```bash
flint helper type newnumber <Type>    # Get next artifact number
flint shard list                      # List installed shards
```

## Skills

| Skill | File | Purpose |
|-------|------|---------|
| Init Update | `sk-f-init_update.md` | Update the Flint Init from current session changes |
| Init Full Update | `sk-f-init_full_update.md` | Research entire Flint and rewrite Init from scratch |
| Send to Inbox | `sk-f-inbox_send.md` | Gather files for a topic and send them to another Flint |
| Process Inbox | `sk-f-inbox_process.md` | Process incoming bundles from `Inbox/` into `Mesh/` |

## Templates

| Template | File | Purpose |
|----------|------|---------|
| Flint Init | `tmp-f-flint_init-v0.1.md` | Structure for `(System) Flint Init.md` |
| Concept Note | `tmp-f-concept-v0.1.md` | Concept note — atomic idea, densely linked |
| Record Note | `tmp-f-record-v0.1.md` | Record note — fact, event, or observation |

## Knowledge

| Knowledge            | File                 | Purpose                                                 |
| -------------------- | -------------------- | ------------------------------------------------------- |
| Template Syntax      | `knw-f-templates.md` | Complete guide to reading and generating from templates |
| Artifact Conventions | `knw-f-artifacts.md` | Frontmatter, tags, naming, notes, session tracking      |
| CLI Reference        | `knw-f-cli.md`       | Flint CLI commands for agent workflows                  |
