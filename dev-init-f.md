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

A Flint is one folder of notes and shards, with its own id, that the `flint` CLI manages. It organizes human knowledge and agent capabilities together. It contains a `Mesh/` & `Media/` content layer and a `Shards/` capabilities layer. A Flint can reference external codebases and resources via `flint.toml`. Everything you read, write, and create lives inside the Flint.

# Mesh

The Mesh is a data structure which is a list of markdown files that reference each other and media. It is the content layer of a Flint — everything under `Mesh/` and `Media/`. It holds typed artifacts (`(Task)`, `(Plan)`, `(Notepad)`, etc.), notes, dashboards, system files, and archives. `Mesh/Types/` organizes artifacts by type into subfolders. `Mesh/Notes/` holds free-form notes. `Mesh/Archive/` stores completed work. Dashboards (`(Dashboard) *.md`) are live DataviewJS views that aggregate artifacts. `Media/` holds non-markdown files (images, PDFs, etc.). All the content of the Flint goes into the Mesh — never write outside of it.

The Mesh is **flat except for section folders**: a document is in zero or one section, and the folder of a sectioned document has meaning — it lives in the folder `(Section) <Name>/` of its section; every other folder is display only. Structure lives in frontmatter tags. The structure layer comes from the **Invironments shard** (`ie`), a dependency of Flint: `#ie/sections/<key>` says which section a node is in (zero or one), `#ie/groups/<key>` says what collections it's part of, and a bare `#ie` marks a `(Section)` header. The Flint shard owns the conventions built on it: `Mesh/Main/` holds the staging sections (**New → Working → Consolidated**; dumped notes land in New), `Mesh/Sections/` holds other section folders at any depth (a section folder is never inside another one), `Mesh/Groups/` holds group definitions. Load the `ie` shard for the full grammar and tooling.

# Shards

Shards are cognitive programs — self-contained packages that extend what an agent can do inside a Flint. A shard has a **source** (the files a person edits) and a **shard** (the built package that agents load). Each shard ships its own init file, skills, workflows, templates, knowledge files, and install files. Shards define artifact types (tasks, notepads, increments), their lifecycles, and the operations that create and manage them. Without shards, a Flint is just an empty folder. With shards, it becomes a structured environment for planning, building, and tracking work.

**Modules.** A module is a package of the kind `module`, as a shard is a package of the kind `shard`. A shard serves agents; a module serves the machine. Module code runs as a process in any language (Node.js, Python, or an executable). A Flint installs a module with `flint module install <spec>`: the record goes to `[modules]` of `flint.toml`, the build goes to `Modules/<Name>/`, and the settings of the module in this Flint go to `Modules/<Name>.settings.toml`. `flint sync` runs the plan of each module and makes its objects real on this machine (cron schedules, stations). `flint module <name> <command>` runs a command of a module. The first-party modules are `crons` (the cron schedules of the Flint) and `dms` (direct messages to the Flint). `flint module list` shows the modules of this Flint. See [[dev-knw-f-cli]] § Modules. To write a module, load the Knap shard and follow its workflow `knap_module`.

**Env files.** `flint.env` holds the env values of the Flint, and `flint.env.local` holds the values of this machine (a value there wins). Git ignores both. `flint.env.example` (tracked) names the values with no values. When a task needs a Flint-level value (a token, a key, an address), read it from `flint.env.local` and `flint.env` at the Flint root. Orbh does not put these values into a session. Never copy a value into the Mesh, a commit, or a message. See [[dev-knw-f-cli]] § Env Files.

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

**Invocations**: a workflow with an invocation file runs as a durable record: `flint shard invoke <ref> <workflow>` opens it, `flint shard step done|skip <step>` settles its steps in order, and each write ends with `next:` (do the first line first). When `invoke` refuses because an artifact is not ready (`input-not-ready`), do not edit the artifact to pass the check: in an interactive session, tell the person the state now, the state that the workflow needs, and each way, and run the way that the person picks; in a headless session, return the refusal as the blocker. `flint shard ready '<artifact>'` lists the workflows that can take an artifact next. Read [[dev-knw-f-invocations]] before you run one.

**Discovering shards**: `flint shard list` shows what this Flint has. `flint shard browse [query]` shows what this Flint can install: the shards of the shard registry and the shard sources of the Flints of this machine, each with its state here and the spec that installs it. `flint resolve <spec>` says where one package is. See Choosing Shards below.

### Source and Shard

Every shard has two entities. The folder name says which one a folder holds:

| Entity | Folder | Editable? | Loaded by |
|--------|--------|-----------|-----------|
| The shard (the build) | `Shards/<Name>/` (the alias as a Title when the alias is not the slug of the name) | No — overwritten by the next build or install | `flint shard start` |
| A remote source | `Shards/(Source Remote) <Name>/` | Yes — a clone of a repository; changes are pushed to origin | `flint shard start-dev` |
| A local source | `Shards/(Source Local) <Name>/` | Yes — no repository | `flint shard start-dev` |

The shard id is `shard.yaml#id`; the source id is `shard.yaml#source.id`. The address is `@org/shard/<slug>` (the slug is the fold of the name), and `@org/<slug>` is its short form. The words are in the glossary of the spec ([[(Spec) Flint Shards#Glossary]]). `flint shard build <alias>` makes the shard from its source and strips the `dev-` prefix. The one exception is the `install/` folder — its contents are literal payloads (dashboards, type definitions, note templates) and carry no `dev-` prefix.

The Flint keeps the shard state in three places: `flint.toml` holds the intent (`<alias> = "<spec>"`, or `{ source = "<spec>", git?, from = "source"?, use? }`); `flint.json#shards[<shard id>]` is the lock (the state: `published`, `snapshot`, or `edited`, with its proof); `.flint/shards.json` holds the facts of this machine. A reference to a shard is a package spec: `@org/name[@version][#place]`.

### Shard File Types

| Pattern | Purpose |
|---------|---------|
| `shard.yaml` | Manifest — name, version, dependencies, setup, types, folders, install |
| `init-[sh].md` | Context — load first, defines shard rules; its `# Headless` section holds the rules of a headless session |
| `setup-[sh].md` | Setup lifecycle — one-time setup instructions for the shard |
| `sk-[sh]-[name].md` | Skill — atomic task, follow actions exactly |
| `wkfl-[sh]-[name].md` | Workflow — multi-stage, human review between stages; `> [!headless]` callouts in a stage say what a headless session does there |
| `wkfl-[sh]-[name].invocation.json` | Invocation file of a workflow — inputs, steps, outputs, behaviours, readiness, and the next workflow; `flint shard invoke` reads it |
| `tmp-[sh]-[name]-v<X.X>.md` | Template — versioned structural guide for creating artifacts |
| `knw-[sh]-[name].md` | Knowledge — deep reference material |
| `ast-[sh]-[name].[ext]` | Asset — non-markdown files |
| `mig-[sh]-<from>-to-<to>.md` | Migration — upgrade script when the version of the shard bumps |
| `inst-[sh]-[name].md` | Install payload (under `install/`) — dashboards, system files, folder anchors; `dest` field in `shard.yaml` carries the literal target filename |
| `otmp-[sh]-[name].md` | Note template (under `install/`) — a Templater template for the Obsidian template picker. It installs into `Shards/(Shards) Obsidian Templates/` with `force` semantics, byte for byte: no `#readonly`, no install-time placeholders. Use Templater syntax: `<% crypto.randomUUID() %>`, `<% tp.date.now("YYYY-MM-DD") %>` |
| `type-[sh]-[type].md` | Type definition (under `install/`) — driven by `types:` field, installs to `Mesh/Metadata/Types/` |
| `scripts/<name>.js` | Script — Node.js command, invoked via `flint shard <sh> <name>` |

Source files add a `dev-` prefix (e.g. `dev-sk-<sh>-<name>.md`). Everything under `install/` is the exception — `inst-`, `otmp-`, `type-` files carry no `dev-` prefix.

### Choosing Shards

The **core shards** are Flint (`@nuucognition/shard/flint`) and Orbh (`@nuucognition/shard/orbh`). Everything else is chosen for what the Flint is for. `flint create` gives every new Flint the core shards, whatever its preset declares, and Invironments (Flint needs it). Then it installs the shards of the preset. The preset `blank` (the default of `flint create`) adds no shards. The preset `default` adds Projects, Notepad, Plan, Increments, Reports, Agents, and Claude Code. `flint create --help` lists the other presets. A clone (`flint create --from <url>`) keeps the shard list of its files. An install from a path or a Git location needs no NUU account and no org. Only a publish to the shard registry (`flint shard release`) needs an org. The rules:

1. **Look before you build.** Before you install a shard and before you create one, run `flint shard browse <word>` (what exists: the shard registry and the shard sources of this machine) and `flint shard list` (what this Flint has). If a shard already provides the capability, install it with the spec in its row. Do not create a duplicate shard. When the shard registry does not answer, `browse` lists only the shard sources of this machine and prints one warning; then also search the site of the shard registry, `https://shards.nuucognition.com/registry`. `flint resolve @org/shard/<slug>` says whether one package is in this Flint, on this machine, or in the shard registry.
2. **New Flint: ask, then suggest.** In the first session of a new Flint, ask the operator what the Flint is for. Then use [[dev-sk-f-shard_browse]] to suggest shards and install the operator's picks with `flint shard install <spec>` (the missing dependencies come too).
3. **On demand later.** When the operator asks for a capability, look first and install the matching shard. Prefer a public shard over a local source; a local source is a working version in another Flint of this machine (install it as `@org/<slug>#<flint>`).
4. **Core missing?** When `flint shard list` has no row for `flint` or `orbh`, run `flint shard install --core`. It installs each core shard that this Flint does not have and leaves the present ones alone.
5. **Load after install.** Run `flint shard start <alias>` and read the init file before you use a new shard.

### Shard Manifest

Each shard has a `shard.yaml` at its root declaring identity, dependencies, and installation behavior:

- `shard-spec`, `shard` (the package name, quoted: `shard: "@nuucognition/flint"`; its slug is the slug of `name`), `id` (the shard id), `source: { id, of? }` (the source id), `version`, `name`, `shorthand`, `description`. A manifest at `shard-spec: "0.3.0"` has `org` (the org slug) in place of `shard`
- `formerNames`, `formerShorthands`, `of` — the rename history and the fork origin, written by the CLI
- `dependencies:` — a map from package name to range (e.g. `"@nuucognition/flint": "^0.2"`)
- `setup:` — `full | flint | local` when the shard needs one-time setup
- `types:` — artifact types the shard manages (installs `(Type) ...` files to `Mesh/Metadata/Types/`)
- `folders:` — artifact storage folders to create under `Mesh/`
- `install:` — files copied verbatim from `install/` into the Flint (dashboards, system files, note templates)

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

Always read `Mesh/(System) Flint Init.md` first. It contains what this Flint is about, how to navigate it, what shards are installed, and the instructions of this Flint.

### Identity

The person who works in a Flint has one **Name** ("your Name" in the screens of the CLI). Your Name is how the person appears as the author of notes on this machine. It is bound to a `Mesh/People/@<Name>.md` file. Your Name is the same for every Flint on this machine: it is stored in the NUU config (`config.toml` in the NUU home, `~/.nuucognition` or `$NUU_HOME`).

`flint setup` sets up this machine in seven steps: your Name and the machine display name (the name that people read, such as "Ada Laptop"), the Git author, a check of the tools, the agent default (the first copy of the Orbh profiles and the Orbh default target list), the Computer Flint (always made), the Guide of NUU Cognition (a public Flint in the home folder that answers Flint questions with an agent; `flint setup guide` clones it alone), and the open of the Computer Flint with the onboarding session. Setup needs internet access: with no network it stops before the first step. `flint setup onboard` starts or resumes the onboarding session alone. The machine name (the short word after `#` in an address) comes from the machine display name, and a rename of the machine keeps it. The branch prefix (`machine-name`) takes its default from the hostname. `flint config name "<Name>"` changes your Name only. `flint config org set <slug>` sets the org of this machine: each new Flint gets it. The id comes from `--id <uuid>`, from the enrollment of this machine, or from a Flint of that org on this machine. `flint config org unset` makes new Flints local again. A NUU account (`flint login`) and an org are optional: Flint works with no NUU account and no org, and no local command checks them.

A Flint with no org is local. Its address starts with `@/` (for example `@/flint/my-notes`). An address that starts with `@/` names no org. It works only on this machine. To share the Flint, give it an org: `flint org set <org>`. The Flint keeps its id.

The **Computer Flint** is the home of the machine: the Flint `(Computer) <Machine Display Name>` in the home folder. The word `computer` is reserved for it: `flint open computer` and `flint cd computer` always reach it, and no other Flint can have `computer` as its name, local name, or alias. It holds the "Start here" note and the repair notes. `flint rename "<name>"` in the Computer Flint, or `flint setup --machine-proper-name "<name>"`, renames the machine and the Computer Flint together.

Once a day, a command in a terminal checks in the background for a newer Flint and newer Orbh profiles; a later command names each with its command (`flint update`, `flint orbh profiles update`). `FLINT_NO_UPDATE_CHECK=1` or `flint config set update-check off` turns it off, and it never runs in an Orbh session. When a Flint command fails, its output ends with `Or: flint fix`. `flint fix` starts a repair session in the Computer Flint with the failure record; the repair session follows [[dev-sk-f-repair]]. `flint doctor` checks the machine and the current Flint and starts no agent.

```bash
flint setup                      # Set up this machine in seven steps; it ends in the Computer Flint with the onboarding session
flint whoami                     # Show your Name, this machine, your org, and the NUU account
flint config name "Your Name"    # Change your Name only
flint create "<name>"            # Create a Flint (init is an alias)
flint doctor                     # Check this machine and the current Flint; one next command per problem
flint fix                        # Start a repair session for the last failure
```

**When creating or editing artifacts**, resolve the Name (run `flint whoami`, or read `name` from the NUU config) and populate the `authors` frontmatter field with the person as a wikilink:

```yaml
authors:
  - "[[@Nathan Luo]]"
```

- The `@Person` is `@<Name>`; `flint create` writes the `Mesh/People/@<Name>.md` marker, and it is auto-created on first use.
- `authors` is a list (supports multiple people collaborating on one artifact)
- If no Name is set, omit the `authors` field (the operator runs `flint setup` to set it)
- Person files live in `Mesh/People/` with the `@` prefix convention

### Session Tracking

When you create or edit a file with frontmatter, stamp your session in the field `orb:`:
- Run `flint address me`. Append its `by` address to `orb:` as a quoted string: `orb: ["@v1/<org>/session/<uuid>#flint/<slug>"]`.
- If the field does not exist, create it. Add nothing only when `orb:` or `orbh-sessions` has an item that is exactly your `by` address, the short form `@v1/<org>/session/<uuid>`, `[[<uuid>]]`, or your bare `<uuid>`. Do not parse other items: append your address.
- Old notes have the field `orbh-sessions` (`"[[<uuid>]]"` items). Never remove it, and add no new item to it.
- All agent types (Claude, Codex, etc.) use the same `orb:` field — there are no per-runtime session fields. See [[dev-knw-f-artifacts]] § Session Tracking.

### Modes

A Flint agent runs in one of two modes: `interactive` (a person at the terminal) or `headless` (an Orbh session with no person: a headless, subagent, or ping session). The mode comes from the Orbh session. One set of files serves both modes:

- Load a shard with `flint shard start <ref>` (`start-dev` for a source) in both modes. In a headless session, also read the section `# Headless` of the init, when it has one: the interface keys, the phase values, and the result delivery.
- A workflow has `> [!headless]` callouts in the stages that differ. In a headless session, do the normal text of each stage and follow its callout; when they conflict, the callout wins. An interactive session ignores the callouts.
- A workflow invocation knows the mode. In the headless mode, it skips each step that only a person can do (`flint shard step` shows the skip), and its hints print the text of the headless mode.
- Removed: `flint shard hstart`, `hstart-dev`, `--headless`, and the headless files `hinit-<sh>.md` and `hwkfl-<sh>-<name>.md` (NUU Orbh Task 748). `hstart` and `hstart-dev` refuse with `hstart-removed` and give the next command `flint shard start '<ref>'` (`start-dev`). `invoke --headless` refuses with `headless-flag-removed` before any write. The CLI reads no headless file, and the health check gives a problem for each one. Use `flint shard start <ref>`. Outside a headless Orbh session, a test uses `--mode headless`.

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
4. **Write outputs to Mesh/** — all the content of the Flint lives under Mesh/
5. **Search, don't browse** — find files by name/tag, not by walking directories
6. **Load shards on demand** — don't load what you don't need

### Available Commands

See [[dev-knw-f-cli]] for the full CLI reference. Most commonly used:

```bash
# Artifact helpers
flint helper artifact create <Type> --title "<Title>"   # Reserve the next number and write the artifact file
flint helper type newnumber <Type>    # Read the next number (a preview: it reserves nothing)
flint helper rename "<old>" "<new>"   # Rename a Mesh artifact + rewrite every [[Old]] wikilink under Mesh/
flint helper delete "<name>"          # Delete artifact + strip every frontmatter wikilink to it (use --archive to soft-delete)

# This machine and new Flints
flint setup                           # Set up this machine in seven steps: identity, Git author, tools, agent default, Computer Flint, Guide, open and onboarding
flint whoami                          # Show your Name, this machine, your org, and the NUU account
flint doctor                          # Check this machine and the current Flint; one next command per problem; starts no agent
flint fix [--list]                    # Start a repair session for the last failure record (--list: the records)
flint config name "<Name>"            # Change your Name only
flint config org set <slug> [--id <uuid>]   # The org of this machine: new Flints get it
flint create "<name>"                 # Create a Flint (init is an alias); --preset <name> adds shards
flint org set <org>                   # Give this Flint an org (a local Flint has none: @/)

# Shard discovery and loading (<ref> = alias, shorthand, address, or id; a former address or a former shorthand resolves with a moved note)
flint shard list                      # One row per shard: id, address, alias, shorthand, version, state
flint shard browse [query]            # What this Flint can install: the shard registry and the shard sources of this machine
flint shard status <ref>              # The row, the Git state of the source, dependencies, pending migrations (info is an alias)
flint shard start <ref>               # Dynamic manifest of the shard (both modes)
flint shard start-dev <ref>           # Dynamic manifest of the source (both modes)
flint shard ready '<artifact>'        # The workflows of the installed shards that can take the artifact, with each verdict
flint resolve <spec>                  # Where a package is: this Flint, this machine, or the shard registry

# Shard lifecycle
flint shard install                   # Make the lock match the specs of flint.toml (pnpm install)
flint shard install @org/name[@range][#place]   # Install a package (the shard registry, or a place)
flint shard install --from-git <owner/repo[#ref]>   # Install from a Git location
flint shard install --from-path <dir>           # Install from a folder on this machine
flint shard install --core            # Install each missing core shard (Flint, Orbh)
flint shard build <ref>               # Build the shard from its source in this Flint
flint shard install --all-dev         # Build the shard of every source
flint shard update [<ref>]            # Move the lock of each shard to the highest version inside its range (pnpm update)
flint shard broadcast <ref>           # After a change to a source here: update the shard in each Flint of this machine that installs it from here
flint shard uninstall <ref>           # Remove the shard and clean files

# Sync and transport
flint sync                            # Make the files match the lock and the declarations (pnpm install --frozen-lockfile)
flint git sync                        # Exchange history with origin; the local sync runs before the push
```

Three commands, one job each. `flint sync` makes the files of this Flint match its lock and its declarations. `flint shard install` makes the lock match the specs. `flint git sync` exchanges history with origin. `flint sync` never asks the shard registry and never touches the origin of the Flint. Its one network action beside the fetch of missing bytes is a `git fetch` of each workspace repository, which it reports and never pulls. It writes no pin and never moves a shard of the registry or of Git to another version; a build of a source or of a path record takes the version of that source or folder (see [[knw-f-cli]] § Sync).

Authoring commands (create, build, dev, clone, release, fork, rename, id, push, pull, migrate, scripts) are documented by the Knap shard — load `Shards/Knap/init-knap.md` when authoring. Knap also covers the authoring of modules: the module contract, both SDKs, and the workflow `knap_module`.


If this Flint is a member of a **Tinderbox**, `flint tinderbox <command>` manages the box from any member. A Tinderbox is a folder that holds many Flints as one box. It has the intent `tinderbox.toml`, the record `tinderbox.json`, and the local folder `.tinderbox/`. Read [[dev-knw-f-tinderbox]] before you run a write command.
