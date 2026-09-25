---
description: "Tinderbox — the box of Flints, its three files, members, repos and connections, the local sync, the transport, the commands, and the safety rules"
orbh-sessions:
  - "[[2ed34533-a12f-4692-a0b1-e88495548403]]"
  - "[[c08435a9-8e1d-4833-adf5-99a2928c5669]]"
---

# Knowledge: Tinderbox

The model of the Tinderbox and the `flint tinderbox` commands. Read this file before you run a `flint tinderbox` command. The command table of [[dev-knw-f-cli]] is the short form. The spec is [[(Spec) Tinderbox System]].

## What a Tinderbox is

A Tinderbox (the box) is a folder that holds many Flints as one unit. Each Flint of the box is a member. The box is not a Flint, and a Flint is a member of at most one box. Most Flints are in no box.

The folder of a box is `(Tinderbox) <name>`. It is a Git repository: the box repo. The box repo tracks `tinderbox.toml`, `tinderbox.json`, and `.gitignore`. It does not track the members, `Repos/`, or `.tinderbox/`. Each member and each repo is its own Git repository, and `.gitignore` of the box lists their folders.

A member is a normal Flint. `flint sync` and `flint git sync` work in a member as in any Flint. The box adds three things: one intent for many Flints, one local sync for all of them, and one transport for all of them.

## The three files

The box has three files, as a Flint has `flint.toml`, `flint.json`, and `.flint/`. No fact lives in two files.

| File | Job | In the box repo | Same job in a Flint |
|---|---|---|---|
| `tinderbox.toml` | The intent of the box: the name, the members, the repos, the connections | Yes | `flint.toml` |
| `tinderbox.json` | The record of the box: its id, its type, its version, its org, and its members with their ids | Yes | `flint.json` |
| `.tinderbox/` | The facts of this machine: the local version, the last sync, the Git journal, the lock, and the backups | No | `.flint/` |

The operator edits `tinderbox.toml`, or a command writes it. Only the commands write `tinderbox.json` and `.tinderbox/`. Do not edit them by hand.

### `tinderbox.toml`

```toml
[tinderbox]
name = "NUU Flints"

[flints]
required = [
  { name = "NUU Flint", source = "https://github.com/nuu/flint-nuu-flint.git", mode = "own" },
  { name = "NUU Docs", source = "registry:NUU Docs", mode = "reference" },
]
optional = [
  { name = "Research", source = "git@github.com:nuu/flint-research.git" },
]

[repos]
required = [
  { name = "Main App", source = "https://github.com/nuu/main-app.git", exposed-to = "all" },
]

[connections]
required = [
  { interconnected = ["NUU Flint", "NUU Docs"] },
  { from = "Research", to = "NUU Flint", kind = "cites" },
]
```

The full schema and the validation rules are in [[(Spec) Tinderbox System . Configuration]]. A validation failure stops a command before its first write. The org is not intent: a `[tinderbox].org` key has no effect.

### `tinderbox.json`

The record has the keys of `flint.json` that have the same meaning, in this order:

```json
{
  "version": "0.7.0",
  "id": "6c2682c9-9217-4bed-afc8-70a87d113ef3",
  "type": "tinderbox",
  "created": "2026-09-21T00:40:44.553Z",
  "org": { "id": "e17b42dd-1726-463c-861b-49d2a833c73e", "slug": "nuu-cognition" },
  "migrations": { "tinderbox-0.1-to-0.7.0": "2026-09-26T00:00:00.000Z" },
  "members": [
    { "id": "591383f0-db03-410a-9e39-b3751ae371d6", "name": "NUU Flint" }
  ]
}
```

| Key | Meaning |
|---|---|
| `version` | The Flint Version that wrote the box files, as in `flint.json`. A CLI refuses to write a record with a newer version; the next command is to update the CLI. |
| `id` | The id of the box. The first command that records the box mints it. It never changes. |
| `type` | Always `"tinderbox"`. |
| `created` | The time of the mint. |
| `org` | `{ id, slug }`. Absent in the local tier. Only `flint tinderbox org set <org> --apply` writes it. |
| `migrations` | The steps that ran on the box files: the step id and the time. `{}` until a step runs. |
| `members` | `{ id, name }` for each member with a known id, in the order of `tinderbox.toml`. A reference member gets its id from `flint.json` at its roster path. |

The record has no `source`, no `mode`, and no repos: they are intent, and they live in `tinderbox.toml`. A write keeps an unknown top-level key. `remove` and `rename` update the record at once, and a member that is not on this machine keeps its id across a rename.

A record of the old shape (`"spec": "0.1"`, with `source` and `mode` for each member) is read for one release. The first write command upgrades it once: it backs up the old bytes to `.tinderbox/backups/`, writes the new record, and records `tinderbox-0.1-to-0.7.0` in `migrations`. `status`, `check`, `git status`, and each `--dry-run` never write, so they never upgrade.

### `.tinderbox/`

```text
.tinderbox/
├── local.json            the local version and the local steps: { "version", "migrations" }
├── sync.json             the time of the last local sync that changed something: { "synced" }
├── git-sync-state.json   the Git journal of the last or interrupted `git sync`
├── git-sync.lock         the lock of a running `git sync`
└── backups/
    ├── tinderbox.toml.<UTC time>
    └── tinderbox.json.<UTC time>
```

The first write command makes the folder: `init`, `start`, `sync`, `import`, `add`, or `org set`.

The box phase of the local sync stamps `sync.json`. It stamps it only when it has no error, it changed something, and every selected required member is on this machine. The box phase inside `git sync` stamps it too. A dry run never stamps it.

`heal` and `rename` back up `tinderbox.toml` before they rewrite it. `dissolve` backs up `tinderbox.toml` and `tinderbox.json` before it removes them. The record upgrade backs up `tinderbox.json`. `remove` writes no backup. A corrupt journal is kept beside the journal as `git-sync-state.json.corrupt-<time>`.

## Members

A member is a Flint that `[flints]` of `tinderbox.toml` declares. Each entry has a `name` and a `source`. It can also have a `mode` and a `type`. The `name` must be the `[flint].name` of the member.

- **Owned member** (`mode = "own"`; the default for a Git URL). The box owns the Flint. The source is a Git URL. The local sync clones it into the box as `(<Type>) <name>`, for example `(Flint) NUU Mesh`. Each machine that has the box clones the same member. When the roster lists a Flint with the name of an absent owned member at another path, the local sync does not clone it. The member is `blocked` with the next command `flint tinderbox import <name>`. `git sync` gives the same block.
- **Reference member** (`mode = "reference"`; the default for `registry:<name>`). The Flint stays at its path on this machine. The local sync finds it in the roster (`~/.nuucognition/places.json`) and wires it. It never clones, copies, or moves it. Every reference spelling works: the name, the folder form, and the address (`@org/flint/<slug>`, `@/flint/<slug>`). When the roster of this machine does not have the Flint, the member is `blocked`. The next command is `flint create <name>`, or `flint register <path>` when the Flint is on disk.
- **Required and optional.** `[flints].required` holds the members that the box needs. `[flints].optional` holds the members that some machines cannot reach. A failed clone of an optional member is a warning, not a failure. Its row says `skipped (optional, not on this machine)`. An optional member with a roster collision is `skipped` with a warning.
- **Type.** `type` (a lowercase slug) is the Flint type of the member. It sets the folder prefix: `type = "computer"` gives `(Computer) <name>`. The default is `flint`.

"On this machine" is the column `PRESENT` of `status`. It means an owned member that is cloned in the box, or a reference member at its roster path.

## Repos and connections

A **repo** is a codebase that is not a Flint and that the box shares with its members. `[repos].required` declares it.

- An owned repo has a Git URL as its source. The local sync clones it into `Repos/<slug>/`.
- A reference repo has `path:<dir>` as its source. It stays at its path.
- `exposed-to` is `"all"` (the default) or a list of member names. Each of these members gets a codebase reference to the repo in its `flint.toml`.

A **connection** is a Flint reference that the local sync wires from one member to another. `[connections].required` declares it.

- `{ from = "A", to = "B" }` gives A a reference to B.
- `{ interconnected = ["A", "B", "C"] }` gives each member of the group a reference to each other member.
- `kind` (optional, a lowercase slug) names the relation.

The local sync writes each reference into `[references]` of `flint.toml` of the member and fulfils it in `.flint/references.json`. `repo remove` and `connection remove` strip the references that they wired. A strip that fails is reported as a failure, and no line says "stripped" for it. A strip that empties a list removes the key.

## Local sync

`flint tinderbox sync` is the local sync of the box. It makes the box match its intent on this machine, then runs `flint sync` in each selected member.

The box phase, in order:

1. Read and validate `tinderbox.toml`. A failure stops the run before the first write.
2. Make the box repo when it is absent, and refresh `.gitignore` of the box.
3. Clone each missing owned repo and each missing owned member.
4. Find each reference member in the roster.
5. Register the members in the roster, and as Obsidian vaults unless `--no-open` is set.
6. Wire the repos and the connections into the members.
7. Record the box in `tinderbox.json`.

Then the member runner runs `flint sync` in each selected member, in this process, 5 at a time.

Rules:

- The local sync never asks the registry, never moves a lock, and never touches origin. The one fetch is `git clone` of an owned member or repo that the intent names and that is absent. `flint sync` has the same exception.
- The local sync never moves or deletes a folder. Undeclared folders stay. `check` and a run with no `--only` and no `--skip` report them.
- `--dry-run` shows the plan of the box (the clones, the references, the wiring) and the plan of each selected member. It writes nothing.
- `--only <names...>` and `--skip <names...>` select members. The repos are synced only in a run with no `--only` and no `--skip`. A run with `--only` or `--skip` reports no undeclared folder.
- In a box with an org, an owned member with no org or another org is a notice. Its next command is `flint tinderbox org set <org> --id <uuid> --apply`. The local sync never writes the org of a member.
- `--json` prints one operation report for the box, with `members[]` (one report for each member). A refusal is one JSON value too.
- `--no-git` and `--no-update` are retired. Each prints one line that names the right command, and the sync runs.

## Transport

`flint tinderbox git sync` is the transport of the box. It exchanges the history of the box repo and of each member with origin, with the local sync in the middle. The order is the order of `flint git sync`:

1. **The box:** checkpoint, fetch, integrate. No push yet. When the box does not integrate, the run stops before the members: a halt exits 2 (`blocked`), and an error exits 1.
2. **The local sync of the box:** the box phase of `flint tinderbox sync`, with no member runner. A member that the pull declared for the first time is cloned here.
3. **The members,** 4 at a time. Each member runs the flow of `flint git sync`: checkpoint, fetch, integrate, the local sync, checkpoint, push. A reference member is `skipped (reference)`: its own Flint moves its history.
4. **The box:** the second checkpoint (it commits the writes of the local sync), then the push.

Rules:

- `--no-sync` is transport only: no local sync of the box and none in the members.
- `--only <names...>` selects members. The box repo always syncs.
- `--force-local` and `--force-remote` choose the strategy for overlapping changes in each member, as in `flint git sync`.
- A record that a newer CLI wrote refuses `git sync` and `git resume` before any Git work. The refusal exits 1 with the next step: update the CLI.
- A conflict in the box repo aborts the rebase of the box, because no command resolves the box repo. The run is `blocked`. The next step is Git in the box repo: `git pull --rebase`, resolve, `git rebase --continue`, then `flint tinderbox git sync`.
- A required member that is still not on this machine after the local sync of the box is a failed row. It keeps the reason and the next command of that local sync. When the roster lists the Flint at another path, the member is `blocked` with `flint tinderbox import <name>`. With `--no-sync`, an absent member is `failed` with the next command `flint tinderbox sync`. An optional member that is not on this machine is `skipped (optional, not on this machine)`.
- A conflict in a member holds the rebase. The member is `blocked` with one next command: `flint tinderbox git resolve <member>`. The text "the rebase was aborted" prints only when a rebase was aborted.
- The journal `.tinderbox/git-sync-state.json` records each member of a run. The lock `.tinderbox/git-sync.lock` stops a second `git sync` of the same box on this machine.
- `git status` shows the journal and the Git state of each member. It writes nothing. A reference member is never `pending`.
- `git resume` continues each member that the last run did not finish. It never advises `resume` again.
- `git resolve <member>` fixes one member. It continues the held rebase, or it syncs the member again with `--local` or `--remote`. Use `--replay` after a remote history rewrite.
- `git publish <url>` sets origin of the box repo, names the branch `main`, commits pending changes, and pushes. It compares the configured URL of origin (`git config --get remote.origin.url`), so an `insteadOf` rule is not a change of origin. It asks unless `--yes`. A declined prompt exits 1, and the next command adds `--yes`.
- Each Git command takes `--json`: one JSON value on every path. The block of members is `Members` and the row of the box is `Box` in each Git command.

## Status, exit codes, and next commands

Each Tinderbox command renders as `flint sync` and `flint git sync` do. It prints one title line with the status, sections with glyphs, and each block once. `dissolve` and `org set` are the exceptions: they print their own text.

`tinderbox sync` and `tinderbox git sync` use one status rule. The rows of the members and the steps of the box give the counts, and the counts give the status:

| Status | Meaning | Exit |
|---|---|---|
| `ok` | Each member is done, current, or skipped | 0 |
| `partial` | Some members failed, and other members succeeded | 1 |
| `failed` | Every member failed, or the command was refused | 1 |
| `blocked` | A block and no failure. Examples: a held rebase, an absent reference member, a roster collision, a halt of the box repo | 2 |

- A step of the box counts only when it changed something, failed, or is blocked. A required member that is still not on this machine is a failed row, not a fatal issue.
- A halt of the box repo is `blocked` and stops the run before the members.
- A skipped row gives its reason in brackets in both verbs, for example `skipped (reference)`.
- `tinderbox sync` exits as `flint sync` does. `tinderbox git sync`, `git resume`, and `git resolve` exit as `flint git sync` does.
- A refusal (no box, an invalid `tinderbox.toml`, a bad argument) exits 1 with the reason. Most refusals give the next command on a `Next` line. Some refusals of the domain give it inside the message.
- `check` counts each finding as failed and exits 1 when it has a finding. A warning and a notice do not count. A required reference member that the roster does not have is a finding. A malformed or newer `tinderbox.json` is a finding.
- `status` exits 0 also when this CLI cannot read `tinderbox.json`. Its title then says `org not known (tinderbox.json cannot be read)`, and a warning gives the next step.
- A dry run marks nothing `synced` and counts nothing `done`. A blocked member in a dry run is `blocked`, not `failed`.
- Each issue gives at most one next command, and the report prints each next command once.
- `--json` prints one JSON value on every path, with `status` and `next`. `status --json` adds `org`, `synced`, `members`, and `repos`.

## The commands

Every command runs from the box root or from any folder in a member (see "Where to run"). A command that writes nothing is marked "reads".

| Command | What it does | Flags | Exit |
|---|---|---|---|
| `init <name>` | Makes `(Tinderbox) <name>` in the current folder with the three files and a first commit. | — | 0, 1 |
| `init --from <url>` | Clones the box from its Git URL into the current folder, then runs the local sync. It prints one report. | `--no-open` | 0, 1 |
| `start <name> [path]` | Makes a new box and moves the current Flint into it as its first owned member. | — | 0, 1 |
| `import <name> [source]` | Moves a Flint of the roster into this box as an owned member, or declares it from `[source]` when the roster does not have it. | `--no-open`, `--yes` | 0, 1 |
| `add <name> <source>` | Declares a member in `tinderbox.toml` with no change on disk: an owned member from a Git URL, or a reference member from `registry:<name>`. | — | 0, 1 |
| `remove <name>` | Removes a member from `tinderbox.toml` and the record, and strips the references that the box wired. The folder stays unless `--move-out` moves it. The roster row stays. | `--move-out <dir>` | 0, 1 |
| `rename <from> <to>` | Renames a member: its declaration, its record entry, its `flint.toml` name, and its folder. | — | 0, 1 |
| `rename --tinderbox <name>` | Renames the box, its folder, and its roster row. | — | 0, 1 |
| `dissolve` | Backs up and removes `tinderbox.toml` and `tinderbox.json`, removes the roster row, and strips the wiring. `.git`, the member folders, and `.tinderbox/` stay. | `--dry-run`, `--move-to <dir>`, `--force`, `--yes` | 0, 1 |
| `sync` | Makes the box match its intent on this machine, then runs `flint sync` in each selected member. | `--dry-run`, `--json`, `--only <names...>`, `--skip <names...>`, `--no-open` | 0, 1, 2 |
| `status` | Reads: the box (its name, its org, its last sync) and the state of each member and connection. | `--json`, `--wide` | 0, 1 |
| `check` | Reads: compares the intent with this machine and gives one next command for each finding. | `--json` | 0, 1 |
| `heal` | Repairs the drift that `check` finds (with a backup of `tinderbox.toml`), then runs the local sync of the box. It exits 1 when a finding stays. | `--dry-run`, `--yes`, `--json` | 0, 1 |
| `repo add <name> <source>` | Declares a repo and clones it into `Repos/`, or references it at `path:<dir>`. | `--exposed-to <all\|names>`, `--mode <own\|reference>` | 0, 1 |
| `repo remove <name>` | Removes a repo and strips its codebase reference from each member. | `--purge`, `--yes` | 0, 1 |
| `repo list` | Reads: the repos, their mode, source, path, members, and whether each is on this machine. | `--wide` | 0, 1 |
| `connection add [from] [to]` | Declares a connection: one direction, or an interconnected group. | `--group <names...>`, `--kind <slug>` | 0, 1 |
| `connection remove [from] [to]` | Removes a connection and strips the references that it wired. | `--group <names...>` | 0, 1 |
| `connection list` | Reads: the connections and whether the local sync wired each one. | `--wide` | 0, 1 |
| `git sync` | Exchanges the history of the box repo and of each member with origin, with the local sync in the middle. | `--no-sync`, `--only <names...>`, `--force-local`, `--force-remote`, `--json` | 0, 1, 2 |
| `git status` | Reads: the Git journal of the last or interrupted run and the Git state of each member. | `--json` | 0, 1 |
| `git resume` | Continues each member that the last `git sync` did not finish. | `--no-sync`, `--json` | 0, 1, 2 |
| `git resolve <member>` | Fixes one blocked member: continues its held rebase, or syncs it again with a strategy. | `--local`, `--remote`, `--replay`, `--no-sync`, `--json` | 0, 1, 2 |
| `git publish <url>` | Sets origin of the box repo, names the branch `main`, commits pending changes, and pushes. | `--yes`, `--json` | 0, 1 |
| `org set <org>` | Prints the plan to set the org of the box and of each owned member, and writes it with `--apply`. | `--id <uuid>`, `--apply`, `--json` | 0, 1 |

## Safety rules

- **The commands that move a folder:**
  - `start` moves the current Flint into the new box.
  - `import` moves a Flint of the roster into the box. It asks unless `--yes`.
  - `remove --move-out <dir>` and `dissolve --move-to <dir>` move member folders out of the box.
  - `rename` moves the member folder. `rename --tinderbox` moves the box folder.
  - `heal` gives a member folder its canonical name. It moves a broken member folder aside as `<folder>.broken-<time>`.
  - `flint move` of a member asks unless `--yes`. It removes the member from `tinderbox.toml`, then moves the folder.
- **The one command that deletes:** `repo remove --purge` deletes the clone under `Repos/`. It asks unless `--yes`.
- **`flint tinderbox sync` never moves or deletes a folder.** It clones what is absent and keeps undeclared folders.
- **The commands that write `tinderbox.toml`:** `init`, `start`, `import`, `add`, `remove`, `rename`, and `heal`. Also `repo add`, `repo remove`, `connection add`, `connection remove`, and `flint move` of a member. `heal` and `rename` back up `tinderbox.toml` to `.tinderbox/backups/` first. `dissolve` backs up `tinderbox.toml` and `tinderbox.json` first.
- **Preview first.** Run `--dry-run` first where it exists: `sync`, `heal`, `dissolve`. `org set` without `--apply` is a plan. The local sync of `heal` registers the members as Obsidian vaults. `heal --dry-run` does not list these registrations, and `heal` has no `--no-open`.
- **The commands that write nothing:** `status`, `check`, `git status`, `repo list`, `connection list`, each `--dry-run`, and `org set` without `--apply`.
- **`dissolve` protects work.** It refuses when a member has uncommitted or unpushed work, unless `--force`. With no terminal to ask, it refuses unless `--yes`.
- **Know which box you are in.** A command walks up to the nearest `tinderbox.toml`. A command in any folder under a box acts on that box, also in a repo under `Repos/`. Never run a write command in a box that you do not own. Scripts and checks run in a scratch box with an isolated home.
- **Use the commands, not the file system.** Do not move, rename, or delete a member folder by hand. Use `remove`, `rename`, or `dissolve`.

## The local tier

- A box with no org is in the local tier. It reads clean: no command asks for an org or an account. Its address is `@/tinderbox/<slug>`.
- `flint tinderbox org set <org> --apply` gives the box and each owned member the org. `--id <uuid>` gives the id when this machine does not know the org yet. Without `--apply` the command prints the plan and changes nothing. A reference member keeps its own org.
- In a box with an org, an owned member with no org or another org is a notice. `sync`, `sync --dry-run`, and `check` give it, with the next command `flint tinderbox org set <org> --id <uuid> --apply`. No command heals it: the local sync never writes the org of a member.
- `status` shows the org of the box in its title, or `no org (local tier)`. A `tinderbox.json` that this CLI cannot read is not the local tier: the title says `org not known (tinderbox.json cannot be read)`.
- A `registry:` source resolves from the roster with every reference spelling, the `@/` address included.
- No Tinderbox command checks the account.

## Where to run

- Each command except `init` and `start` walks up from the current folder to the nearest `tinderbox.toml`. So it runs from the box root or from any folder in a member.
- With no `tinderbox.toml` above the current folder, a command refuses with one text and the next command.
- A folder that holds both `tinderbox.toml` and `flint.toml` is refused. The refusal names `flint.toml`.
- `init` makes the box in the current folder. It refuses in a Flint (the next command is `flint tinderbox start`) and in a box.
- `start` runs in the Flint that becomes the first member. The Flint must have a Git origin.

## See also

- [[dev-knw-f-cli]] — the command table of the Tinderbox, with all other Flint commands.
- [[(Spec) Tinderbox System]] — the spec: Configuration, Discovery, Sync Protocol, Health, CLI, Move.
- [[Guide - Using Tinderbox]] — the guide for the operator.
- [[(Spec) Flint State Contract]] — what `flint.toml`, `flint.json`, and `.flint/` hold.
