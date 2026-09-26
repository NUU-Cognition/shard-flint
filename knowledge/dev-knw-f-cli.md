---
description: "Flint CLI commands for agent workflows"
orbh-sessions:
  - "[[bd0b60bc-bca8-4b9c-8c66-ee82ccbfb5bb]]"
  - "[[87347e6b-2363-4434-9a98-f1d641049fa7]]"
  - "[[97c9f9ca-46fb-41d8-a7c1-c1a1e099f8b9]]"
  - "[[2ed34533-a12f-4692-a0b1-e88495548403]]"
  - "[[c08435a9-8e1d-4833-adf5-99a2928c5669]]"
  - "[[96f34e4f-b89e-4f22-b801-38f3ad1668fe]]"
  - "[[6ca3f8d9-0148-433a-b327-1d355a60a895]]"
  - "[[ec731b56-92e3-425a-9d01-bed0e4fc3365]]"
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
- Scope is `Mesh/` only. Files under `Shards/`, `Inbox/`, `Archive/`, etc. are not touched.
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
- Scope is `Mesh/` only. `Shards/`, `Inbox/`, `Archive/`, etc. are not touched.
- Errors if `<name>` is not found.

Also available over HTTP as `POST /api/artifacts/by-title/<name>/delete?archive=true|false`.

## Identity and New Flints

`flint setup` sets the identity of this machine once: your Name and the machine display name. The machine name (the short word after `#` in an address) comes from the machine display name. The branch prefix (`machine-name`) takes its default from the hostname. Your Name and the machine names are in the NUU config (`config.toml` in the NUU home, `~/.nuucognition` or `$NUU_HOME`); every NUU CLI reads them. `flint config org set <slug>` sets the org of this machine: each new Flint gets it. The id comes from `--id <uuid>`, from the enrollment of this machine, or from a Flint of that org on this machine. `flint config org unset` makes new Flints local again. A NUU account and an org are optional.

```bash
flint setup                              # Set your Name and the machine names (once per machine)
flint whoami                             # Show your Name, this machine, your org, and the NUU account
flint config name "<Name>"               # Change your Name only
flint config org set <slug> [--id <uuid>]   # The org of this machine: new Flints get it
flint create "<name>"                    # Create a Flint (init is an alias)
flint create "<name>" --preset default   # Create a Flint with the shards of a preset
flint create --from <url>                # Clone a Flint from a Git repository
flint org show                           # The org and the address of this Flint
flint org set <org>                      # Give this Flint an org; its id does not change
```

A Flint with no org is local. Its address starts with `@/` (for example `@/flint/my-notes`). An address that starts with `@/` names no org. It works only on this machine. To share the Flint, give it an org: `flint org set <org>`.

## Shard Discovery

A shard is a package with two entities: the **source** (the files a person edits, in `Shards/(Source Local) <Name>/` or `Shards/(Source Remote) <Name>/`) and the **shard** (the build in `Shards/<Name>/`, or the alias as a Title when the alias is not the slug of the name). `flint.toml` is the intent, `flint.json#shards[<shard id>]` is the lock, and the shard registry gives the versions of each package. The words are in the glossary of the spec ([[(Spec) Flint Shards#Glossary]]).

A `<ref>` names one shard of this Flint. One resolver reads it in this order: the alias (its key in `flint.toml`), the shorthand, the address (`@org/<slug>` or `@org/shard/<slug>`), the id (`<uuid>`, `@<uuid>`, or a prefix of eight or more characters), then a former address (`@org/<former slug>` or `@org/shard/<former slug>`) or a former shorthand. A former form prints `moved: <old> is now <new>` and goes on. A bare word is an alias or a shorthand; a name is an address. The name, a former name, a bare former slug, and a folder name are not a ref: use the alias, or the address for a former name. A ref that names two shards is refused as `ambiguous`, with one next command per shard that names its alias.

```bash
flint shard list [--json]             # One row per shard: ID ADDRESS ALIAS SHORTHAND VERSION STATE
flint shard status <ref> [--json]     # The row, the Git state of the source, dependencies, pending migrations
flint shard status <ref> --health     # Also run the health check (it writes nothing)
flint shard info <ref>                # An alias of status
flint resolve <spec>                  # The answer of the walk: this Flint, this machine, or the shard registry
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

## Shard Browsing

`flint shard browse` shows what this Flint can install. Run it before you install a shard and before you create one. It reads only; it writes nothing.

```bash
flint shard browse [query]            # The catalog; the query matches the name, slug, shorthand, address, or description
flint shard search <query>            # The same command (search is an alias of browse)
flint shard browse --available        # Only the shards that this Flint does not have
flint shard browse --installed        # Only the shards that this Flint has
flint shard browse --json             # One value: { query, entries, warnings }
```

The catalog has two kinds of entries. A **public** entry comes from the shard registry; its spec is `@org/<slug>`. A **local** entry is a shard source (`Shards/(Source Remote) <Name>/` or `Shards/(Source Local) <Name>/`) in a Flint of this machine; its spec is `@org/<slug>#<flint slug>`. One shard can have several rows: one for the registry and one for each Flint that holds its source. The columns are `ADDRESS VERSION FROM STATE SPEC NEXT`:

- `FROM` is `registry`, `<Flint> (remote source)`, `<Flint> (local source)`, or `here (…)` for a source in this Flint.
- `STATE` is `installed <version> (<state>)`, `not installed`, or `source here`. The state comes from the lock of this Flint, matched by the shard id, then the address.
- `NEXT` is `flint shard install <spec>` for a shard that is not installed, and `flint shard build <alias>` for a source here with no build.

When the shard registry does not answer, `browse` lists the shard sources of this machine and prints one warning with the reason. Then also search the site of the shard registry, `https://shards.nuucognition.com/registry`. `flint resolve @org/shard/<slug>` says whether one package is in this Flint, on this machine, or in the shard registry. If a shard already provides the capability, install it instead of writing a duplicate. Prefer a public shard over a local source; a local source is a working version in another Flint of this machine.

**Core shards** are Flint (`@nuu-cognition/shard/flint`) and Orbh (`@nuu-cognition/shard/orbh`). Every new Flint gets them, whatever its preset declares; a clone keeps the shard list of its files. When `flint shard list` has no row for one of them, install it:

```bash
flint shard install --core            # Install each missing core shard; leave the present ones alone
```

`--core` installs a missing core shard through the normal install path (its dependencies first, then the lock). It reads the lock by shard id, address, or Git location, never by folder. It takes no other source: with a spec, `--from-git`, `--from-path`, or `--all-dev` it refuses and names the two commands to run. `--json` gives `{ ok, shards, present }`. The core shards install from their Git locations (`NUU-Cognition/shard-flint`, `NUU-Cognition/shard-orbh`), because the shard registry does not serve them as `@nuu-cognition/<slug>` yet.

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

A package spec is `@org/name[@version][#place]`: `@version` is an exact version, a caret range (`^1.1`), or a tilde range (`~1.1.3`); `#place` is a machine name or the slug of a Flint on this machine. Without `#place`, the install asks this Flint, this machine, then the shard registry.

```bash
flint shard install                              # No argument: make the lock match the specs of flint.toml
flint shard install @org/name[@range][#place]    # From the shard registry (published), or from a place
flint shard install --from-git <owner/repo[#ref]>  # From a Git location; the shard registry names the state
flint shard install --from-path <dir>            # From a folder on this machine
flint shard install --core                       # Each missing core shard (Flint, Orbh)
flint shard install <input> --alias <alias>      # Install under another key (a second shard with the same slug)
flint shard install <input> --reference          # No build under Shards/; the loader reads the folder in its place
flint shard install <input> --no-deps            # Do not install the missing dependencies first
flint shard build <ref>               # Build the shard from its source in this Flint
flint shard install --all-dev         # Build the shard of every source
flint shard reinstall [<ref>]         # Install the shard again from its record (a source record builds)
flint shard update [<ref>]            # Move the lock of each shard to the highest version inside the range of its spec
flint shard uninstall <ref>           # Remove the build, its lock record, and its unchanged payload files
```

`flint shard install` with no argument reads every record of `flint.toml`. A record that the lock does not satisfy (no lock line, or the range, the place, or the Git location changed) is installed or built, after its missing dependencies. A record that the lock satisfies is not moved: only `flint shard update` moves a version inside its range. `flint shard update` installs the missing dependencies of the new version first. The pins of the shard repositories (`flint.json#repos`) are lock data: only `flint shard install` with no argument and `flint shard update` write them. Then one registry read per record with a hash records the registry answer and follows a rename that the registry reports. When the registry does not answer, the notice is `The registry did not answer. The answers were not recorded.` `--dry-run` shows the plan and the answers and writes nothing.

```
$ flint shard install @nuu-cognition/meeting-notes@^0.1
✓ Resolved @nuu-cognition/shard/meeting-notes 0.1.0
✓ Installed Meeting Notes 0.1.0
  Spec    : @nuu-cognition/meeting-notes@^0.1
  Address : @nuu-cognition/shard/meeting-notes
  State   : published 0.1.0
  Registry: published
```

An install writes the record `<alias> = "<spec>"` in `flint.toml` (no id), the lock record `flint.json#shards[<id>]` with the state, and the local entry in `.flint/shards.json`. From the registry it checks the shard id and the package hash of the version before any write. It installs the missing dependencies first, with one plan line each. It runs every refusal of the root shard before the first write, also before the dependency installs: the alias or the shorthand is taken, the shard is already present, a present dependency is outside its range, or the id or the hash differs from the registry. When the registry does not answer, an install from Git or from a path goes on and the lock says `registry: unchecked`.

The package hash has two rules: the current rule, and the rule of the builds before commit `5d606f91`, which included the root documents (`README.md`, `RELEASE.md`, `MIGRATIONS.md`). Every compare accepts either rule, and every write uses the current rule. `flint shard uninstall` also removes the old root documents of a build from before `5d606f91`.

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

`setup` reads the setup scope that the start gate reads. So `--complete` also completes a manifest with the legacy key `state: true`.

## Shard Migrations

```bash
flint shard migrate list <ref>        # List the migration steps of a shard
flint shard migrate run <ref>         # Run the queued steps (stops at an agent or manual step)
flint shard migrate run <ref> --dry-run   # Print the steps and the rewrite plan; write nothing
flint shard migrate finish <ref>      # Mark the current agent or manual step done
```

A step with a `rewrite` block (a shorthand rename) rewrites the tags, the links, and the command texts of the Mesh as code, prints one line per file, and then stops at the agent step: follow [[dev-sk-f-migrate]]. `run`, `finish`, and `rerun` hold the lock of the shard while they run, as an install does.

## Shard Scripts

Shards can ship Node.js scripts under `scripts/`, auto-discovered and invoked via the alias or the shorthand of the shard:

```bash
flint shard scripts <ref>             # List executable scripts for a shard
flint shard <ref> <script> [args...]  # Run a declared script
```

> Authoring commands (`create`, `build`, `type add`, `id`, `rename`, `fork`, `clone`, `dev`, `push`, `pull`, `release`, `unpublish`, `published`) are documented by the Knap shard — load `Shards/Knap/init-knap.md` when authoring shards. `publish` is a deprecated alias of `release`: it prints one line and runs the release path, with the same arguments and the same result.

## Sync

Three commands, one job each:

- `flint sync` makes the files of this Flint match its lock and its declarations.
- `flint shard install` makes the lock match the specs.
- `flint git sync` exchanges history with origin.

Shards are packages. `flint shard install` is `pnpm install`: a spec that the lock does not satisfy is resolved, and a satisfied lock is not moved. `flint shard update` is `pnpm update`. `flint sync` is `pnpm install --frozen-lockfile`.

```bash
flint sync                            # Make the files match the lock and the declarations
flint sync --dry-run                  # Show the plan; write nothing (the same as flint plan)
flint shard install                   # Make the lock match the specs; record the registry answers
flint shard install --dry-run         # Show the plan and the registry answers; write nothing
flint git sync                        # Exchange history with origin; the local sync runs before the push
flint git sync --no-sync              # Transport only: no local sync
flint git sync --continue             # Continue a sync halted on a conflict (a bare re-run also continues)
flint git sync --abort                # Stop the held rebase and restore the branch
flint git sync --replay               # After a remote history rewrite: replay the local-only commits
flint git sync --force-remote         # Overlapping changes take the remote side; a backup ref keeps the old tip
flint git sync --force-local          # Overlapping changes take the local side
flint git merge                       # The flow of git sync with no push [--replay --force-remote --force-local --no-sync --json]
flint git resolve --local|--remote [paths...]   # During a conflict halt, take one side of each file whole
flint git init                        # Make the repository, or refresh the full policy: .gitignore, .gitattributes, rerere
flint git publish <url>               # Set origin, commit, and push to main (the Flint must be a repository root)
flint git <git args...>               # Passthrough to git at the Flint root
```

`flint sync` has two steps: `Resolve Flint` and `Reconcile content`. It never asks the registry, and it never touches origin. It writes no shard version and no pin: it checks out the pin of each shard repository, and it fetches only a pinned commit that is absent. Its one write to the lock removes a stale lock line: a line that `flint.toml` does not declare and that has no folder under `Shards/`. It fetches only bytes that the lock or a declaration names and that are absent or differ: a locked build, the Obsidian payload pin, a declared source that is not cloned, a repository clone, a source repository snapshot. `flint sync --dry-run` and `flint plan` open no socket. A halted rebase blocks the run (exit 2) with the next command `flint git sync --continue`. A held merge, cherry-pick, or revert blocks it too: finish it with `git <kind> --continue` or stop it with `git <kind> --abort`, then run `flint sync`. A `flint.toml` that does not parse blocks the run (exit 2): correct the file, then run `flint sync`. When the branch has commits that the cached ref `origin/<branch>` does not have, sync gives the notice `Local history is N commits ahead of origin. Run flint git sync.` A branch with no `origin/<branch>` ref (never pushed) gives no notice. The flags `--no-git` and `--no-update` are retired: each prints one line that names the new command, and the sync runs.

A declared repository, a workspace repository, and a bundle each have a name that is one folder name. Sync checks that each folder is inside its parent folder before it writes. A reference to a Flint that is not on this machine is one report row, not a failure. The next command for a broken codebase reference is `flint fulfill codebase <name> <path>`. `flint reference codebase`, `flint reference flint`, and `flint reference remove` write the declaration only and print `Next: flint sync`: the sync fulfils the reference.

For shards, sync runs two reconciles. **The shard reconcile** (feature `shards`) makes each build match the lock: it installs a missing locked build from the lock, builds a stale build of a source again, fetches the locked version when the build differs from the lock, follows a rename that the source, the place, or the build shows (`moved: <Old> is now <New> (<address>); the folder, the key, and the type files followed`; the id stays), refreshes the path of a reference, and makes the missing Mesh folders of `folders:` in the manifest. A record with no lock line is not current (`not-locked`), with the next command `flint shard install`. A shard repository with no pin is `not-locked` too. A lock from a place heals from that place only. When that place holds a changed source, the record is `not-locked`, with the next command `flint shard update <alias>`. **The source reconcile** (feature `shard-sources`) records the Git state of each source and gives a notice for a source that is a draft, behind, or dirty. It makes the missing spec folders of a source (`skills/`, `scripts/`, `migrations/`, …), because Git keeps no empty folder. It never writes a file into a source. `flint shard clone` makes the same folders after the clone. A missing dependency or a dependency outside its range is not current, with the next command. The rename process is in [[(Spec) Flint Shards . Rename]]. On a Flint with the 0.6.0 shard records, `list`, `status`, and `start` read them with a notice, and every shard write is refused with the next command `flint migrate run`.

`flint git sync` checkpoints the local work, fetches, integrates, runs the local sync, checkpoints the writes of the sync, and pushes. A conflict halts before the local sync: resolve it, then run `flint git sync --continue`. `flint git merge` does the same with no push. Each next command of a merge uses the verb `merge`: after a conflict, run `flint git merge` again; after a remote history rewrite, run `flint git merge --replay`. `--continue` with `--replay`, `--force-remote`, or `--force-local` is refused. `flint git resolve` takes paths relative to the working directory. A held merge, cherry-pick, or revert refuses `sync` and `merge` with exit 2, and the run changes nothing. The next command is `git <kind> --continue`; then run the command again. One Git run at a time holds the run lock of the repository, `<git dir>/flint-git-sync.lock`: `sync`, `merge`, `--continue`, `--abort`, and `resolve` take it. When the wait for the lock times out (60 seconds), the report is `blocked` with exit 2. When the local sync fails, the push still sends the integrated history as it is, with no second fetch; when origin moved during that sync, the push is rejected, and the next `flint git sync` sends it. A Git halt prints under `Failed` and exits 2. `--json` prints the report as one JSON value.

`flint git init` in an existing repository refreshes the full policy: `.gitignore`, `.gitattributes`, and `rerere.enabled`. `flint git publish <url>` refuses a folder that is not the root of a repository, with the next command `flint git init`. The passthrough (`flint git <git args...>`) gives the terminal to git, so an editor and `add -p` work. A network command of the passthrough keeps the network timer of the other Git commands.


## Workspace

```bash
flint workspace                       # The repositories of this Flint (codebases, URLs)
```

## Tinderbox

A Tinderbox (the box) holds many Flints as one unit. Each Flint of the box is a member.

`tinderbox.toml` is the intent of the box: the name, the members, the repos, the connections. `tinderbox.json` is the record of the box: its id, its type, its version, its org, its members with their ids, and the references that the box wired (`wired`). `.tinderbox/` holds the facts of this machine: the local version, the last sync, the Git journal, the lock, and the backups. No fact lives in two files.

`flint tinderbox sync` is the local sync of the box. It makes the box match its intent on this machine. It clones a missing owned member, references a roster member, registers the members, wires the connections and the repos, and records the box. The clone is the one fetch: bytes that the intent names and that are absent, the same exception that `flint sync` has. Then it runs the local `flint sync` in each selected member. It never asks the registry, never moves a lock, and never touches the origin of a member. `--dry-run` shows the plan and writes nothing. The box removes only the references that it wired, so a reference that a person added stays.

`flint tinderbox git sync` is the transport of the box. It exchanges the history of the box repo and of each member with origin, with the local sync in the middle. The order is the order of `flint git sync`: checkpoint, fetch, integrate, local sync, checkpoint, push. `--no-sync` is transport only. The Git journal and the lock of a run live in `.tinderbox/`. A reference member is `skipped (reference)`: its own Flint moves its history.

Read [[dev-knw-f-tinderbox]] for the model, the write gate, the exit codes, and the safety rules before you run a write command.

```bash
# The box and its members
flint tinderbox init <name>                   # Make (Tinderbox) <name> here with the three files and a first commit [--no-open: only with --from]
flint tinderbox init --from <url>             # Clone a box from Git, then run the local sync; exit 2 when a member is blocked [--no-open]
flint tinderbox start <name> [path]           # Make a new box and move the current Flint into it as an owned member
flint tinderbox import <name> [source]        # Move a Flint of the roster into the box, or declare it from [source] [--no-open --yes]
flint tinderbox add <name> <source>           # Declare a member without cloning or moving it: a Git source (owned) or registry:<name> (reference) [--json]
flint tinderbox remove <name>                 # Remove a member and keep its folder, unless --move-out moves it [--move-out <dir> --json]
flint tinderbox rename <from> <to>            # Rename a member: declaration, record, flint.toml name, folder, roster row, vault, wired references; backs up both box files first [--json]
flint tinderbox rename --tinderbox <name>     # Rename the box, its folder, its roster row, and the member vaults; backs up both box files first [--json]
flint tinderbox dissolve                      # Remove the box and keep the member folders; backs up both box files first [--dry-run --move-to <dir> --force --yes --json]

# The local sync and the health of the box
flint tinderbox sync                          # Make the box match its intent, then run flint sync in each selected member [--dry-run --json --only <names...> --skip <names...> --no-open]
flint tinderbox status                        # The box (name, org, last sync) and each member and connection [--json --wide]
flint tinderbox check                         # Compare the intent with this machine; one next command per finding [--json]
flint tinderbox heal                          # Repair what check finds (with a backup of the box files), then run the local sync of the box; exit 1 when a finding stays [--dry-run --yes --json --no-open]

# Repos and connections
flint tinderbox repo add <name> <source>      # Declare a repo: clone a Git source into Repos/, or reference path:<dir> [--exposed-to <all|names> --mode <own|reference> --json]
flint tinderbox repo remove <name>            # Remove a repo and strip its codebase references [--purge --yes --json]
flint tinderbox repo list                     # The repos and whether each is on this machine [--wide]
flint tinderbox connection add [from] [to]    # Declare a connection: one direction, or a group [--group <names...> --kind <slug> --json]
flint tinderbox connection remove [from] [to] # Remove a connection and strip the references that it wired [--group <names...> --json]
flint tinderbox connection list               # The connections and whether each is wired (alias: connections) [--wide]

# The transport
flint tinderbox git sync                      # Exchange the box and member histories with origin, with the local sync in the middle [--no-sync --only <names...> --force-local --force-remote --json]
flint tinderbox git status                    # The Git journal and the Git state of each member; writes nothing [--json]
flint tinderbox git resume                    # Continue each member that the last git sync did not finish [--no-sync --json]
flint tinderbox git resolve <member>          # Fix one blocked member: continue its rebase, or sync it again; finish a held merge with git first [--local --remote --replay --no-sync --json]
flint tinderbox git publish <url>             # Set origin of the box repo (the configured URL is compared), name the branch main, commit, and push; with --json or no terminal it needs --yes [--yes --json]

# The org
flint tinderbox org set <org>                 # The plan for the org of the box and of each owned member; --apply writes it; none removes the org [--id <uuid> --apply --json]
```

## Send / Inbox

```bash
flint send <target> <title> <instructions> <files...>  # Send files to another Flint's inbox as a bundle
flint send "My Other Flint" "Auth Migration" "Move tasks to Tasks/" file1.md file2.md
```

Files land in the target Flint's `Inbox/(Bundle) Title/` directory. Use [[dev-sk-f-inbox_process]] to process incoming bundles.

## Other Useful Commands

```bash
flint open                            # Open a Flint in Obsidian
```
