---
description: "Flint CLI commands for agent workflows"
orbh-sessions:
  - "[[87347e6b-2363-4434-9a98-f1d641049fa7]]"
---

# Knowledge: CLI Reference

Quick reference for Flint CLI commands relevant to agent workflows.

## Artifact Helpers

```bash
flint helper type newnumber <Type>    # Next number for a typed artifact (zero-padded, 3 digits)
flint helper type newnumber Task      # → 119
flint helper type newnumber Notepad   # → 034
```

## Session Markdown Pins

Use pins to show important Markdown files first in the current session's finder. Open the finder with `Ctrl-]` then `o`.

```bash
flint helper pins set "Mesh/Types/Tasks/(Task) 687 Example.md" "Mesh/Notes/Design.md"
flint helper pins add "Mesh/Notes/Decision.md"
flint helper pins remove "Mesh/Notes/Decision.md"
flint helper pins list
flint helper pins clear
flint helper pins list --session <id> --path /path/to/flint
```

The helper uses `ORBH_SESSION_ID` unless you pass `--session`. Use `--orb-root` or `--at` for an explicit store.
It follows the current session's workspace after a shell directory change. It rejects a target from another Flint.

`set` replaces the list in argument order. `add` appends new paths and removes duplicates. `remove` accepts missing files.
`list` prints JSON. `clear` writes an empty list. Quote each filename, including its extension.

Paths are relative to the Flint root. Absolute paths within that root also work.
Additions must be readable `.md` or `.markdown` files of at most 2 MiB.
Hidden paths, symbolic links, and dependency or build directories are outside the finder scope.
The helper checks all additions before it writes. An invalid addition leaves the list unchanged.

The session interface key `flint:markdown-pins` holds a JSON array of path strings. It is separate from `artifacts`.
The helper uses the normal session setter, including manager routing and durable events.
Pins stay with the session across resume and compaction. Child sessions start with their own list.

The finder shows matching pins first in stored order. Other matches keep the usual fuzzy ranking.
It refreshes pins while open. A missing file remains visible as unavailable.
Renames do not update pins automatically. Remove the old path and add the new path after a rename.
Malformed interface values produce an error without a write.

Pins belong to the displayed session in Flint attach, interactive, and explicit transcript or detail views.
A general cockpit list or picker has no pin owner. Standalone Orbh, Strike, and dashboards do not use these pins.

Task workflows use `add` when `ORBH_SESSION_ID` is set. Keep completed tasks pinned until the session changes the list.
Do not pin every file that an agent reads or edits.

## Rename

**Always use this** when renaming a Mesh artifact — it renames the file **and** rewrites every `[[Old Name]]` wikilink (including `[[Old|alias]]`, `[[Old#heading]]`, and `![[Old]]` embeds) under `Mesh/` in one pass. Don't do manual `mv` + grep/sed for artifact renames.

```bash
flint helper rename "<old>" "<new>"   # Rename by title; Unix-mv arg order (old first, new second)
flint helper rename "(Task) 050 Smoke Old" "Smoke New"
flint helper rename "(Task) 050 Smoke Old" "(Task) 050 Smoke New"   # Full new name also accepted
```

Behavior:
- Looks up `<old>` by Mesh title (Mesh names are globally unique — no ambiguity).
- Strips any `(Type) ` / `(Type) NNN ` / `(Type) X.N - ` prefix from `<new>`, keeping the prefix from the old filename.
- Scope is `Mesh/` only. Files under `Shards/`, `Inbox/`, `Exports/`, `Archive/`, etc. are not touched.
- Errors if the destination already exists or `<old>` is not found.

Also available over HTTP as `POST /api/artifacts/by-title/<old>/rename` with body `{"title": "<new>"}`.

## Delete

**Always use this** when deleting a Mesh artifact — it removes the file **and** strips every wikilink to it from YAML frontmatter across `Mesh/` (scalar fields like `parent:` get their key removed; array fields like `artifact-refs:` get filtered, and the key is removed if the array becomes empty). **Body text is left untouched by design** — a dangling `[[Name]]` in prose is a harmless informational trace, not a structural invariant.

```bash
flint helper delete "<name>"                # Hard delete (rm) + frontmatter sweep
flint helper delete "<name>" --archive      # Move to Mesh/Archive/ instead of hard-deleting
flint helper delete "(Task) 042 Old Plan"
```

Behavior:
- Looks up `<name>` by Mesh title (Mesh names are globally unique).
- Scope is `Mesh/` only. `Shards/`, `Inbox/`, `Exports/`, `Archive/`, etc. are not touched.
- Errors if `<name>` is not found.

Also available over HTTP as `POST /api/artifacts/by-title/<name>/delete?archive=true|false`.

## Identity

```bash
flint whoami                             # Show current person identity
```

## Shard Discovery

A shard is a package with two entities: the **source** (the files a person edits, in `Shards/(Source Local) <Name>/` or `Shards/(Source Remote) <Name>/`) and the **shard** (the build in `Shards/<Name>/`, or the alias as a Title when the alias is not the slug of the name). `flint.toml` is the intent, `flint.json#shards[<shard id>]` is the lock, and the NUU Shard Registry gives the versions of each package. The words are in the glossary of the spec ([[(Spec) Flint Shards#Glossary]]).

A `<ref>` names one shard of this Flint. One resolver reads it in this order: the alias (its key in `flint.toml`), the shorthand, the address (`@org/<slug>` or `@org/shard/<slug>`), the id (`<uuid>`, `@<uuid>`, or a prefix of eight or more characters), then a former address (`@org/<former slug>` or `@org/shard/<former slug>`) or a former shorthand. A former form prints `moved: <old> is now <new>` and goes on. A bare word is an alias or a shorthand; a name is an address. The name, a former name, a bare former slug, and a folder name are not a ref: use the alias, or the address for a former name. A ref that names two shards is refused as `ambiguous`, with one next command per shard that names its alias.

```bash
flint shard list [--json]             # One row per shard: ID ADDRESS ALIAS SHORTHAND VERSION STATE
flint shard status <ref> [--json]     # The row, the Git state of the source, dependencies, pending migrations
flint shard status <ref> --health     # Also run the health check
flint shard info <ref>                # An alias of status
flint resolve <spec>                  # The answer of the walk: this Flint, this machine, or the registry
```

```
$ flint shard list
Shards
  ID                ADDRESS                             ALIAS          SHORTHAND   VERSION     STATE
  4426ad5b          @nuu-cognition/shard/meeting-notes  meeting-notes  meet        0.1.0       edited

  1 shard, 1 declared
```

`status` prints one row with the labels `Id`, `Address`, `Alias`, `Shorthand`, `Name`, `Version`, `State`, `Request`, `From`, `Registry`, `Folders`, in this order. `Version` is the version of the build here; when the source moved past its build, the row says `stale`.

`--json`: `list` gives `{ rows: <row>[] }`; `status` gives `{ ...<row>, details, moved?, health? }`. A row is `{ id, held, alias, shorthand, name, address, request, state, version, from, registry, use, folders: { shard?, source? }, setup, pending, stale }`.

## Shard Browsing (not in this build of the CLI)

`flint shard browse` and `flint shard install --core` are not in this build of the CLI (planned). `flint shard browse` answers `not-found` (the CLI reads `browse` as a ref), and `flint shard install --core` answers `unknown option '--core'`. Until they exist, use these commands before you install a shard and before you create one:

```bash
flint shard list                      # What this Flint has
flint resolve @org/shard/<slug>       # Where one package is: this Flint, this machine, or the registry
flint shard install @org/<slug>       # Install a package and its missing dependencies
```

The public shards are on the registry site: `https://shards.nuucognition.com/registry` (search by name or description). If a shard already provides the capability, install it instead of writing a duplicate. A local source in another Flint of this machine is a working version; install it as `@org/<slug>#<flint slug>`.

**Core shards** are Flint (`@nuu-cognition/flint`) and Orbh (`@nuu-cognition/orbh`). `flint init` installs the shards of its preset. When `flint shard list` has no row for one of them, install it:

```bash
flint shard install @nuu-cognition/orbh
```

## Shard Manifests (loading shards)

`start` / `hstart` assemble a dynamic manifest from each shard's files (init, skills, workflows, templates, knowledge — read from each file's `description` frontmatter). Run the variant that matches your mode. `hstart` loads `hinit-<sh>.md` and its required reading in place of `init-<sh>.md`; a shard with no headless init refuses `hstart` with the next command `flint shard start <ref>`.

```bash
flint shard start <ref>               # The shard, interactive (loads init-<sh>.md)
flint shard start-dev <ref>           # The source, interactive (loads dev-init-<sh>.md)
flint shard hstart <ref>              # The shard, headless (loads hinit-<sh>.md, lists hwkfl-*)
flint shard hstart-dev <ref>          # The source, headless
```

The header names the shard: `# Shard: <Name> (<sh>) v<version>`, then `Id`, `Address`, `Alias`, `State` (`published <tag>`, `snapshot <sha>`, or `edited`), and `Request`. A shard bound by reference (`use = "reference"`) loads from the folder in its place. `start` of a shard whose source changed after the build prints `The build of <alias> is stale: its source changed after the build. Next: flint shard build <alias>` on stderr and loads the build.

The start refuses and exits 1 when setup is required (it prints `FORCE SETUP`, the setup file, and the `SETUP REQUIRED` banner), when shard migrations are pending, or when a reference source is gone. With `--json` the output is one JSON value, also on a refusal (`{ ok: false, code, reason, next }`).

## Shard Install / Update

A package spec is `@org/name[@version][#place]`: `@version` is an exact version, a caret range (`^1.1`), or a tilde range (`~1.1.3`); `#place` is a machine name or the slug of a Flint on this machine. Without `#place`, the install asks this Flint, this machine, then the registry.

```bash
flint shard install @org/name[@range][#place]    # From the registry (published), or from a place
flint shard install --from-git <owner/repo[#ref]>  # From a Git location; the registry names the state
flint shard install --from-path <dir>            # From a folder on this machine
flint shard install <input> --alias <alias>      # Install under another key (a second shard with the same slug)
flint shard install <input> --reference          # No build under Shards/; the loader reads the folder in its place
flint shard install <input> --no-deps            # Do not install the missing dependencies first
flint shard build <ref>               # Build the shard from its source in this Flint
flint shard install --all-dev         # Build the shard of every source
flint shard reinstall [<ref>]         # Install the shard again from its record (a source record builds)
flint shard update [<ref>]            # Move each shard to the highest version inside the range of its spec
flint shard uninstall <ref>           # Remove the build, its lock record, and its unchanged payload files
```

```
$ flint shard install @nuu-cognition/meeting-notes@^0.1
✓ Resolved @nuu-cognition/shard/meeting-notes 0.1.0
✓ Installed Meeting Notes 0.1.0
  Spec    : @nuu-cognition/meeting-notes@^0.1
  Address : @nuu-cognition/shard/meeting-notes
  State   : published 0.1.0
  Registry: published
```

An install writes the record `<alias> = "<spec>"` in `flint.toml` (no id), the lock record `flint.json#shards[<id>]` with the state, and the local entry in `.flint/shards.json`. From the registry it checks the shard id and the package hash of the version before any write. It installs the missing dependencies first, with one plan line each. It refuses before any write when the alias or the shorthand is taken, when a present dependency is outside its range, or when the id or the hash differs from the registry. When the registry does not answer, an install from Git or from a path goes on and the lock says `registry: unchecked`.

## Shard Versioning

```bash
flint shard versions <ref>            # The versions in the registry, else the Git tags of its location
flint shard install @org/<slug>@<version>   # An exact version in the spec: the version does not move
```

The spec carries the version. The retired inputs (`flint://`, `--from-local`, `install --edit`, the version commands of 0.6.0, `edit = true`) each print one refusal with the new spelling; see [[(Spec) Flint Shards . Lifecycle]] § Retired Inputs.

## Shard Setup

```bash
flint shard setup <ref>               # Show the setup state (Flint layer and local layer)
flint shard setup <ref> --complete    # Mark setup complete
flint shard setup <ref> --reset       # Back to required
```

## Shard Migrations

```bash
flint shard migrate list <ref>        # List the migration steps of a shard
flint shard migrate run <ref>         # Run the queued steps (stops at an agent or manual step)
flint shard migrate run <ref> --dry-run   # Print the steps and the rewrite plan; write nothing
flint shard migrate finish <ref>      # Mark the current agent or manual step done
```

A step with a `rewrite` block (a shorthand rename) rewrites the tags, the links, and the command texts of the Mesh as code, prints one line per file, and then stops at the agent step: follow [[dev-sk-f-migrate]].

```bash
```

## Shard Scripts

Shards can ship Node.js scripts under `scripts/`, auto-discovered and invoked via the alias or the shorthand of the shard:

```bash
flint shard scripts <ref>             # List executable scripts for a shard
flint shard <ref> <script> [args...]  # Run a declared script
```

> Authoring commands (`create`, `build`, `type add`, `id`, `rename`, `fork`, `clone`, `dev`, `push`, `pull`, `release`, `unpublish`, `published`) are documented by the Knap shard — load `Shards/Knap/init-knap.md` when authoring shards. `publish` is a deprecated alias of `release`.

## Sync

```bash
flint sync                            # Sync all shards and mods from flint.toml
flint sync --dry-run                  # Show the plan; write nothing
```

For shards, sync runs two reconciles. **The shard reconcile** (feature `shards`) makes each build match the lock: it installs a missing shard, builds a stale build of a source again, fetches the locked version when the build differs from the lock, follows a rename (`moved: <Old> is now <New> (<address>); the folder, the key, and the type files followed`; the id stays), refreshes the path of a reference, and records a changed registry answer with a notice. **The source reconcile** (feature `shard-sources`) records the Git state of each source and gives a notice for a source that is a draft, behind, or dirty; it never changes a source. A missing dependency or a dependency outside its range is not current, with the next command. The rename process is in [[(Spec) Flint Shards . Rename]]. On a Flint with the 0.6.0 shard records, `list`, `status`, and `start` read them with a notice, and every shard write is refused with the next command `flint migrate run`.


## Workspace

```bash
flint workspace                       # Manage workspace references (codebases, URLs)
```

## Tinderbox

A **Tinderbox** is a directory that orchestrates multiple Flints (and shared codebases) as one declared, git-versioned unit. Its `tinderbox.toml` lists member Flints, shared repos, and connections; `flint tinderbox` materializes that declaration on the current machine. **This workspace IS a Tinderbox** — these commands operate on the box that contains it.

- **Members** are declared in `[flints]` in one of two modes. **own** = the Tinderbox owns the Flint and clones it from a git URL into a folder under the box root. **reference** = the Flint lives in the global registry (`registry:Name`); sync points at the existing copy rather than cloning. Members may be `required` (a missing one is an error) or `optional` (a missing one is only a warning — for remotes a given machine cannot reach).
- **Repos** (`[repos]`) are non-Flint codebases shared across the box. Same own/reference modes (clone into `Repos/` vs point at a local path). Each repo is exposed to `all` member Flints or a named subset, and surfaces in each exposed Flint as a codebase reference.
- **Connections** (`[connections]`) declare which Flints reference each other — directional edges (`from -> to`) or interconnected groups. Sync wires these as references inside the child Flints.

All `flint tinderbox` subcommands walk up from the current directory to find `tinderbox.toml`, so they run from the box root or from inside any member Flint.

```bash
# Lifecycle
flint tinderbox init <name>                  # Bootstrap an empty Tinderbox here
flint tinderbox init --from <url>            # Clone an existing Tinderbox from git, then sync (name comes from its toml)
flint tinderbox start <name> [path]          # Promote the current Flint into a new Tinderbox (moves it under the box)
flint tinderbox import <name> [source]       # MOVE a registered Flint into this box (or declare by URL if unregistered)
flint tinderbox add <name> <source>          # Declare a Flint by git URL or registry:Name WITHOUT materializing (sync clones/references it)
flint tinderbox remove <name>                # Eject a member: undeclare + strip wiring, keep it registered (--move-out <dir> relocates the folder)
flint tinderbox rename <from> <to>           # Rename a member Flint (cascades connections/exposed-to/folder/registry)
flint tinderbox rename --tinderbox <name>    # Rename the Tinderbox itself
flint tinderbox dissolve                     # Tear down the box: eject every member (folders survive), strip wiring, remove tinderbox.toml

# Sync, status, drift
flint tinderbox sync                         # Materialize members, register them in Obsidian, wire connections + repos
flint tinderbox status                       # Per-member: mode, tier, present, registered, connection fulfillment (+ Repos)
flint tinderbox check                        # Detect drift between tinderbox.toml and disk (read-only; exits 1 on drift)
flint tinderbox heal                         # Auto-fix drift (rewrites tinderbox.toml) then run a full sync — previews + confirms first

# Repos and connections
flint tinderbox repo add <name> <source>     # Clone a git repo into Repos/ (or path:<dir> reference) and expose it (--exposed-to, --mode)
flint tinderbox repo remove <name>           # Drop a repo declaration + strip child references (--purge deletes the clone)
flint tinderbox repo list                    # List declared repos: mode, location, present, exposed-to
flint tinderbox connection add <from> <to>   # Declare a directional connection (or --group <names...> for an interconnected set)
flint tinderbox connection remove <from> <to> # Remove a connection and strip the wired child references
flint tinderbox connection list              # List declared connection edges and whether each is wired

# Git and identity across the box
flint tinderbox git sync                     # Sync the Tinderbox repo and run `flint git sync` in every member Flint
flint tinderbox git publish <url>            # Add a remote and push the box's initial commit (renames branch to main; --yes skips confirm)
flint tinderbox whoami <name>                # Propagate the person identity to every member Flint
```

**Sync flags:** `--dry-run` (preview clones/moves/deletes, change nothing), `--json` (machine-readable result/plan), `--full` (also run `flint sync` inside every member), `--only <names...>` / `--skip <names...>` (operate on a subset of declared members this run), `--yes` (accept prompts; deletes undeclared Flints that have saved work — needs `--force` for unsaved), `--delete-undeclared` (delete on-disk Flints not in the toml without prompting), `--no-open` (skip Obsidian registration). `status`/`check` also accept `--json`; `heal` accepts `--dry-run` and `--yes`.

**Safety behaviors to know before running these:**
- `sync` **materializes and can MOVE Flints on disk** — it clones missing own-mode members, and adopts/relocates a registry-matched member into the box (after verifying its git remote matches the declared source, with a prompt). `import` likewise **physically relocates** the Flint into the box (confirmed unless `--yes`).
- `sync` can offer to **delete undeclared Flints** (on disk but not in the toml). It guards Flints with uncommitted/unpushed/no-remote work and prompts per-Flint; `--yes`/`--delete-undeclared` automate it, and `--force` is required to delete unsaved work.
- `check` is read-only and reports drift (undeclared/missing/renamed members, broken materialization, stale registry/connections/repos). `heal` is the active repair — it **rewrites `tinderbox.toml` and runs a full sync**, so it previews the plan and confirms before applying (`--dry-run` to stop at the preview).

## Send / Inbox

```bash
flint send <target> <title> <instructions> <files...>  # Send files to another Flint's inbox as a bundle
flint send "My Other Flint" "Auth Migration" "Move tasks to Tasks/" file1.md file2.md
```

Files land in the target Flint's `Inbox/(Bundle) Title/` directory. Use [[dev-sk-f-inbox_process]] to process incoming bundles.

## Other Useful Commands

```bash
flint open                            # Open flint in configured applications
flint export                          # Manage exports
```
