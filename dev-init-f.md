---
required-reading:
  - knowledge/dev-knw-f-models.md
  - knowledge/dev-knw-f-artifacts.md
  - knowledge/dev-knw-f-templates.md
  - knowledge/dev-knw-f-types.md
  - knowledge/dev-knw-f-cli.md
---

# Flint Agent Environment

You are a terminal-based AI agent operating inside a Flint. 

# Flint

A Flint is a workspace — a directory managed by the Flint CLI that organizes human knowledge and agent capabilities together. It contains a `Mesh/` & `Media/` content layer and a `Shards/` capabilities layer. A Flint can reference external codebases and resources via `flint.toml`. Everything you read, write, and create lives inside the Flint.

# Mesh

The Mesh is a data structure which is a list of markdown files that reference each other and media. It is the content layer of a Flint — everything under `Mesh/` and `Media/`. It holds typed artifacts (`(Task)`, `(Plan)`, `(Notepad)`, etc.), notes, dashboards, system files, and archives. `Mesh/Types/` organizes artifacts by type into subfolders. `Mesh/Notes/` holds free-form notes. `Mesh/Archive/` stores completed work. Dashboards (`(Dashboard) *.md`) are live DataviewJS views that aggregate artifacts. `Media/` holds non-markdown files (images, PDFs, etc.). All workspace content goes into the Mesh — never write outside of it.

The Mesh is **flat** — folders are display only; structure lives in frontmatter tags. The structure layer comes from the **Invironments shard** (`ie`), a dependency of Flint: `#ie/sections/<name>` says where a node lives (exactly one mesh section), `#ie/groups/<name>` says what collections it's part of, and a bare `#ie` marks a `(Section)` header. Flint owns the workspace conventions built on it: `Mesh/Main/` holds the staging sections (**New → Working → Consolidated**; dumped notes land in New), `Mesh/Sections/` holds other sections at arbitrary display depth, `Mesh/Groups/` holds group definitions. Load the `ie` shard for the full grammar and tooling.

# Shards

Shards are cognitive programs — self-contained packages that extend what an agent can do inside a Flint. A shard has a **source** (the files a person edits) and a **shard** (the built package that agents load). Each shard ships its own init file, skills, workflows, templates, knowledge files, and install files. Shards define artifact types (tasks, notepads, increments), their lifecycles, and the operations that create and manage them. Without shards, a Flint is just an empty workspace. With shards, it becomes a structured environment for planning, building, and tracking work.

## Shard Rules

Shards extend your capabilities. Each shard is a self-contained unit with its own context, skills, templates, workflows, and knowledge. **You must follow these rules strictly.**

1. **Load before use.** Always read a shard's init file before using any of its skills, workflows, or templates. No exceptions.
2. **On-demand only.** Never load all shards. Only init shards relevant to the current task.
3. **Init gives you the rules.** A shard's init file defines what the shard does, its lifecycle, its file types, and how to use them. Read it fully.
4. **Skills are atomic.** Skills are single-purpose tasks with no human checkpoints. Follow the skill's actions exactly.
5. **Workflows have stages.** Workflows are multi-step tasks with human review points between stages. Do not skip stages.
6. **Knowledge is reference.** Knowledge files contain deep reference material. Read them when you need detailed understanding.

### Loading a Shard

Run `flint shard start <ref>` (the alias or the shorthand) to get the shard's manifest — it lists the init file, required reading, skills, workflows, templates, and knowledge files with descriptions. Read the init file and required reading files before using any capabilities. `start` loads the shard (the build). When the source changed after the build, `start` prints a notice with `flint shard build <alias>` and loads the build anyway.

```
@Shards/[Name]/init-[sh].md                        # Load context (ALWAYS first)
@Shards/[Name]/skills/sk-[sh]-[name].md            # Use a skill
@Shards/[Name]/workflows/wkfl-[sh]-[name].md       # Use a workflow
@Shards/[Name]/templates/tmp-[sh]-[name]-v<X.X>.md # Read a template (versioned)
@Shards/[Name]/knowledge/knw-[sh]-[name].md        # Read knowledge
```

**Sources** live at `Shards/(Source Remote) <Name>/` or `Shards/(Source Local) <Name>/` and prefix every source file with `dev-` (e.g. `dev-init-<sh>.md`, `dev-sk-<sh>-<name>.md`). Load a source with `flint shard start-dev <ref>` only when you edit it. See Source and Shard below.

**Discovering shards**: `flint shard list` shows what this Flint has. The registry site (`https://shards.nuucognition.com/registry`) shows the public shards, and `flint resolve <spec>` says where one package is. `flint shard browse` is not in this build of the CLI (planned). See Choosing Shards below.

### Source and Shard

Every shard has two entities. The folder name says which one a folder holds:

| Entity | Folder | Editable? | Loaded by |
|--------|--------|-----------|-----------|
| The shard (the build) | `Shards/<Name>/` (the alias as a Title when the alias is not the slug of the name) | No — overwritten by the next build or install | `flint shard start` |
| A remote source | `Shards/(Source Remote) <Name>/` | Yes — a clone of a repository; changes are pushed to origin | `flint shard start-dev` |
| A local source | `Shards/(Source Local) <Name>/` | Yes — no repository | `flint shard start-dev` |

The shard id is `shard.yaml#id`; the source id is `shard.yaml#source.id`. The address is `@org/shard/<slug>` (the slug is the fold of the name), and `@org/<slug>` is its short form. The words are in the glossary of the spec ([[(Spec) Flint Shards#Glossary]]). `flint shard build <alias>` makes the shard from its source and strips the `dev-` prefix. The one exception is the `install/` folder — its contents are literal payloads (dashboards, type definitions, Obsidian templates) and carry no `dev-` prefix.

The Flint keeps the shard state in three places: `flint.toml` holds the intent (`<alias> = "<spec>"`, or `{ source = "<spec>", git?, from = "source"?, use? }`); `flint.json#shards[<shard id>]` is the lock (the state: `published`, `snapshot`, or `edited`, with its proof); `.flint/shards.json` holds the facts of this machine. A reference to a shard is a package spec: `@org/name[@version][#place]`.

### Shard File Types

| Pattern | Purpose |
|---------|---------|
| `shard.yaml` | Manifest — name, version, dependencies, setup, types, folders, install |
| `init-[sh].md` | Interactive context — load first, defines shard rules |
| `hinit-[sh].md` | Headless init — loaded in Orbh sessions instead of `init-[sh].md` |
| `setup-[sh].md` | Setup lifecycle — one-time setup instructions for the shard |
| `sk-[sh]-[name].md` | Skill — atomic task, follow actions exactly |
| `wkfl-[sh]-[name].md` | Workflow — multi-stage, human review between stages |
| `hwkfl-[sh]-[name].md` | Headless workflow — used in headless Orbh sessions |
| `tmp-[sh]-[name]-v<X.X>.md` | Template — versioned structural guide for creating artifacts |
| `knw-[sh]-[name].md` | Knowledge — deep reference material |
| `ast-[sh]-[name].[ext]` | Asset — non-markdown files |
| `mig-[sh]-<from>-to-<to>.md` | Migration — upgrade script when the version of the shard bumps |
| `inst-[sh]-[name].md` | Install payload (under `install/`) — dashboards, system files, folder anchors; `dest` field in `shard.yaml` carries the literal target filename |
| `otmp-[sh]-[name].md` | Obsidian template (under `install/`) — human-facing template for the Obsidian picker |
| `type-[sh]-[type].md` | Type definition (under `install/`) — driven by `types:` field, installs to `Mesh/Metadata/Types/` |
| `scripts/<name>.js` | Script — Node.js command, invoked via `flint shard <sh> <name>` |

Source files add a `dev-` prefix (e.g. `dev-sk-<sh>-<name>.md`). Everything under `install/` is the exception — `inst-`, `otmp-`, `type-` files carry no `dev-` prefix.

### Choosing Shards

The **core shards** are Flint (`@nuu-cognition/flint`) and Orbh (`@nuu-cognition/orbh`). Everything else is chosen for what the Flint is for. A new Flint gets the shards of its preset. The preset `blank` (the default of `flint create`) gives Flint and Orbh. The preset `default` gives Flint, Invironments, Projects, Notepad, Plan, Increments, Reports, Agents, and Claude Code, and no Orbh. An install from a path or a Git location needs no NUU account and no org. Only `flint shard publish` needs an org (Task 1035, WP4). `flint shard browse` and `flint shard install --core` are not in this build of the CLI (planned). Until they exist, the rules use `flint shard list` and the registry site. The rules:

1. **Look before you build.** Before you install a shard and before you create one, run `flint shard list` (what this Flint has) and search the registry site `https://shards.nuucognition.com/registry` (what exists). `flint resolve @org/shard/<slug>` says whether a package is in this Flint, on this machine, or in the registry. If a shard already provides the capability, install it. Do not create a duplicate shard.
2. **New Flint: ask, then suggest.** In the first session of a new Flint, ask the operator what the Flint is for. Then use [[dev-sk-f-shard_browse]] to suggest shards and install the operator's picks with `flint shard install <spec>` (the missing dependencies come too).
3. **On demand later.** When the operator asks for a capability, look first and install the matching shard. Prefer a public shard over a local source; a local source is a working version in another Flint of this machine (install it as `@org/<slug>#<flint>`).
4. **Core missing?** `flint init` installs the shards of its preset. When `flint shard list` has no row for `flint` or `orbh`, install the missing one: `flint shard install @nuu-cognition/flint` or `flint shard install @nuu-cognition/orbh`.
5. **Load after install.** Run `flint shard start <alias>` and read the init file before you use a new shard.

### Shard Manifest

Each shard has a `shard.yaml` at its root declaring identity, dependencies, and installation behavior:

- `shard-spec`, `id` (the shard id), `org` (the org slug of the package), `source: { id, of? }` (the source id), `version`, `name`, `shorthand`, `description`
- `formerNames`, `formerShorthands`, `of` — the rename history and the fork origin, written by the CLI
- `dependencies:` — a map from package name to range (e.g. `"@nuu-cognition/flint": "^0.2"`)
- `setup:` — `full | flint | local` when the shard needs one-time setup
- `types:` — artifact types the shard manages (installs `(Type) ...` files to `Mesh/Metadata/Types/`)
- `folders:` — artifact storage folders to create under `Mesh/`
- `install:` — files copied verbatim from `install/` into the workspace (dashboards, system files, Obsidian templates)

Read `shard.yaml` when you need to understand what a shard installs or depends on.

## Template Rules

Templates define how to create artifacts. When you encounter an artifact with a `template` field in frontmatter, or when a skill/workflow tells you to use a template, **follow these rules strictly.**

1. **Find and read the template first.** Search for `tmp-[sh]-[name]-v<X.X>.md` in the shard's `templates/` folder. The `template` field in an artifact's frontmatter is the filename stem (e.g. `[[tmp-proj-task-v0.2]]`).
2. **Templates are instructions, not scaffolds.** They tell you what to generate. Do not copy them verbatim.
3. **Replace all placeholders.** Generated text has no brackets. Follow the instruction inside each placeholder.
4. **Never output comments.** `/* */` blocks are for your understanding only.
5. **Respect the structure.** The template defines sections, ordering, and what fields to include.

## Agent Environment

- **Working directory**: Flint root (parent of `Mesh/`, `Shards/`, etc.)
- **Capabilities**: Read/write files, run shell commands, search codebase
- **Context window**: Limited — load files on demand, never assume prior knowledge
- **Session**: Stateless — each conversation starts fresh
- **Return to root**: At the end of every turn, `cd` back to the Flint root (`cd $ROOT`). Commands may take you into codebases or subdirectories — always return so the next turn starts from a predictable location.

### First Action

Always read `Mesh/(System) Flint Init.md` first. It contains what this Flint is about, how to navigate it, what shards are installed, and workspace-specific instructions.

### Identity

A Flint has a **person identity** — the operator's Name, bound to a `Mesh/People/@<Name>.md` file. The Name is **machine-global**, owned by the nuu CLI and stored in the shared `~/.nuucognition/config.toml` (the same value for every Flint on this machine). Set it once via first-run `flint setup`.

The Name and the machine names (`machine-name` and the machine proper name) come from `flint setup`, which calls `nuu setup` (Task 1035, WP2 and WP3). Login (`nuu login`) and an org are optional: Flint works with no NUU account and no org, and no local command checks them. A Flint with no org has the address `@/flint/<slug>`; `flint org set <org> --id <uuid>` gives it an org later and keeps its id.

```bash
flint whoami                     # Show the operator Name + machine-name (and account status)
flint config name "Your Name"    # Set the operator Name (routes to nuu)
```

**When creating or editing artifacts**, resolve the operator Name (run `flint whoami`, or read `name` from `~/.nuucognition/config.toml`) and populate the `authors` frontmatter field with the person as a wikilink:

```yaml
authors:
  - "[[@Nathan Luo]]"
```

- The `@Person` is `@<Name>`; `flint create` writes the `Mesh/People/@<Name>.md` marker (Task 1035, WP3), and it is auto-created on first use.
- `authors` is a list (supports multiple people collaborating on one artifact)
- If no Name is set, omit the `authors` field (run `flint setup` to set it)
- Person files live in `Mesh/People/` with the `@` prefix convention

### Session Tracking

When editing files with frontmatter, append your session ID to the `orbh-sessions` field:
- `orbh-sessions: [your-session-id]`
- If the field doesn't exist, create it. If it exists, append your ID.
- All agent types (Claude, Codex, etc.) use the same `orbh-sessions` field — there are no per-runtime session fields.

### Headless Mode

A Flint agent can run headless inside an Orbh session (no interactive terminal). In that mode:

- Run `flint shard hstart <ref>` (`hstart-dev` for a source). It loads `hinit-<sh>.md` and its required reading, in place of `init-<sh>.md`.
- `hstart` lists each `hwkfl-<sh>-<name>.md` in place of the interactive `wkfl-*` of the same name, and it lists a `wkfl-*` that has no headless variant. Headless workflows drop human stage gates and report progress via Orbh session keys instead.
- A shard with no headless init refuses `hstart` with the next command `flint shard start <ref>`. Run that command and read the interactive init. There is no automatic fallback.

Headless orchestration details (session lifecycle, status, interface keys, returning results) are owned by the Orbh shard — load `Shards/Orbh/init-foh.md` when operating in that context.

### Deleting Artifacts

**Always use `flint helper delete "<name>"`** when removing a Mesh artifact. Never `rm` the file directly or rely on Obsidian's delete.

What it does:
- Removes the file from disk.
- **Sweeps every YAML frontmatter wikilink** to that artifact across `Mesh/`: scalar fields (e.g. `parent:`, `increment:`) lose their key; array fields (e.g. `artifact-refs:`, `orbh-sessions:`) get filtered, and the key is removed if the array becomes empty.
- **Leaves body text alone by design** — a dangling `[[Name]]` in prose is an informational trace, not a structural invariant.

```bash
flint helper delete "<name>"              # Hard delete + frontmatter sweep
flint helper delete "<name>" --archive    # Soft delete: move to Mesh/Archive/ instead of removing
```

Rules:
- Looks up by Mesh title (Mesh names are globally unique).
- Scope is `Mesh/` only — `Shards/`, `Inbox/`, `Archive/` are untouched.
- Errors if the name is not found — verify the title before retrying.

See [[dev-knw-f-cli]] for the full CLI reference.

### Operating Principles

1. **Read before writing** — understand existing patterns before making changes
2. **Follow naming conventions** — use existing patterns for file names, tags, and structure
3. **Tag documents appropriately** — every artifact gets proper tags
4. **Write outputs to Mesh/** — all workspace content lives under Mesh/
5. **Search, don't browse** — find files by name/tag, not by walking directories
6. **Load shards on demand** — don't load what you don't need

### Available Commands

See [[dev-knw-f-cli]] for the full CLI reference. Most commonly used:

```bash
# Artifact helpers
flint helper type newnumber <Type>    # Next artifact number (zero-padded, 3 digits)
flint helper rename "<old>" "<new>"   # Rename a Mesh artifact + rewrite every [[Old]] wikilink under Mesh/
flint helper delete "<name>"          # Delete artifact + strip every frontmatter wikilink to it (use --archive to soft-delete)

# Identity
flint whoami                          # Show operator Name + machine-name (and account status)

# Shard discovery and loading (<ref> = alias, shorthand, address, or id; a former address or a former shorthand resolves with a moved note)
flint shard list                      # One row per shard: id, address, alias, shorthand, version, state
# flint shard browse, flint shard install --core: not in this build of the CLI (planned)
flint shard status <ref>              # The row, the Git state of the source, dependencies, pending migrations (info is an alias)
flint shard start <ref>               # Dynamic manifest of the shard (interactive)
flint shard start-dev <ref>           # Dynamic manifest of the source (interactive)
flint shard hstart <ref>              # Dynamic manifest of the shard (headless)
flint shard hstart-dev <ref>          # Dynamic manifest of the source (headless)
flint resolve <spec>                  # Where a package is: this Flint, this machine, or the registry

# Shard lifecycle
flint shard install                   # Make the lock match the specs of flint.toml (pnpm install)
flint shard install @org/name[@range][#place]   # Install a package (the registry, or a place)
flint shard install --from-git <owner/repo[#ref]>   # Install from a Git location
flint shard install --from-path <dir>           # Install from a folder on this machine
flint shard build <ref>               # Build the shard from its source in this Flint
flint shard install --all-dev         # Build the shard of every source
flint shard update [<ref>]            # Move the lock of each shard to the highest version inside its range (pnpm update)
flint shard uninstall <ref>           # Remove the shard and clean files

# Sync and transport
flint sync                            # Make the files match the lock and the declarations (pnpm install --frozen-lockfile)
flint git sync                        # Exchange history with origin; the local sync runs before the push
```

Three commands, one job each. `flint sync` makes the files of this Flint match its lock and its declarations. `flint shard install` makes the lock match the specs. `flint git sync` exchanges history with origin. `flint sync` never asks the registry, never moves the lock, and never touches origin.

Authoring commands (create, build, dev, clone, release, fork, rename, id, push, pull, migrate, scripts) are documented by the Knap shard — load `Shards/Knap/init-knap.md` when authoring.


If this Flint is a member of a **Tinderbox** (a folder that holds many Flints as one box, with the intent `tinderbox.toml`, the record `tinderbox.json`, and the local folder `.tinderbox/`), `flint tinderbox <command>` manages the box from any member. Read [[dev-knw-f-tinderbox]] before you run a write command.
