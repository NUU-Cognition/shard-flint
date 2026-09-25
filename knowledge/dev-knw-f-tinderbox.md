---
description: "Tinderbox — the box of Flints, its three files, members, repos and connections, the local sync, the transport, the commands, and the safety rules"
orbh-sessions:
  - "[[2ed34533-a12f-4692-a0b1-e88495548403]]"
  - "[[c08435a9-8e1d-4833-adf5-99a2928c5669]]"
  - "[[96f34e4f-b89e-4f22-b801-38f3ad1668fe]]"
  - "[[6ca3f8d9-0148-433a-b327-1d355a60a895]]"
---

# Knowledge: Tinderbox

The model of the Tinderbox and the `flint tinderbox` commands. Read this file before you run a `flint tinderbox` command. The command table of [[dev-knw-f-cli]] is the short form. The spec is [[(Spec) Tinderbox System]].

## What a Tinderbox is

A Tinderbox (the box) is a folder that holds many Flints as one unit. Each Flint of the box is a member. The box is not a Flint, and a Flint is a member of at most one box. Most Flints are in no box.

The folder of a box is `(Tinderbox) <name>`. It is a Git repository: the box repo. The box repo tracks `tinderbox.toml`, `tinderbox.json`, and `.gitignore`. It does not track the members, `Repos/`, or `.tinderbox/`. Each member and each repo is its own Git repository. `.gitignore` of the box lists the folder of each owned member, `Repos/*/`, `.tinderbox/`, and `/.tinderbox-*/` (the staging folders of a clone or an import). It does not list a reference member, because a reference member is outside the box.

A member is a normal Flint. `flint sync` and `flint git sync` work in a member as in any Flint. The box adds three things: one intent for many Flints, one local sync for all of them, and one transport for all of them.

## The three files

The box has three files, as a Flint has `flint.toml`, `flint.json`, and `.flint/`.

`tinderbox.toml` is the intent of the box: the name, the members, the repos, the connections. `tinderbox.json` is the record of the box: its id, its type, its version, its org, its members with their ids, and the references that the box wired. `.tinderbox/` holds the facts of this machine: the local version, the last sync, the Git journal, the lock, and the backups. No fact lives in two files. One name is in two files on purpose: the member name is in `tinderbox.toml` and in `members[]` of `tinderbox.json`, because it joins a member to its id.

| File | In the box repo | Same job in a Flint |
|---|---|---|
| `tinderbox.toml` | Yes | `flint.toml` |
| `tinderbox.json` | Yes | `flint.json` |
| `.tinderbox/` | No | `.flint/` |

The operator edits `tinderbox.toml`, or a command writes it. A command write changes only the declarations that it edits: the comments and the layout of the file stay. A section that the write empties goes. Only the commands write `tinderbox.json` and `.tinderbox/`. Do not edit them by hand.

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
| `version` | The Flint Version that wrote the box files, as in `flint.json`. A record with a newer version refuses every write command (see "The write gate"). |
| `id` | The id of the box. The first command that records the box mints it. It never changes. |
| `type` | Always `"tinderbox"`. |
| `created` | The time of the mint. |
| `org` | `{ id, slug }`. Absent for a local box (no org). Only `flint tinderbox org set <org> --apply` writes it. |
| `migrations` | The steps that ran on the box files: the step id and the time. `{}` until a step runs. |
| `members` | `{ id, name }` for each member with a known id, in the order of `tinderbox.toml`. A reference member gets its id from `flint.json` at its roster path. |
| `wired` | `{ from, to }` for each reference from one member to another that the local sync wired. The box removes only these references. A write keeps an edge only when both of its ends are declared, and a rename moves it. |

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

Each command that writes the record makes the folder when it is absent: `init`, `start`, `sync`, `import`, `add`, `remove`, `rename`, `rename --tinderbox`, `heal`, `org set --apply`, and `git sync`.

The box phase of the local sync stamps `sync.json`. It stamps it only when it has no error, it changed something, and every selected required member is on this machine. The box phase inside `git sync` stamps it too. A dry run never stamps it.

`rename` and `heal` back up `tinderbox.toml` and `tinderbox.json` before they rewrite them. `dissolve` backs up both files before it removes them. The record upgrade backs up `tinderbox.json`. `remove` writes no backup file. A corrupt journal is kept beside the journal as `git-sync-state.json.corrupt-<time>`.

The Git run writes the journal and the lock only in a real folder `.tinderbox/`. A `.tinderbox` that is a link is refused.

### The write gate

Every command that writes a box file runs one write gate before its first effect. The box files are `tinderbox.toml`, `tinderbox.json`, `.gitignore`, and `.tinderbox/`. The gate reads the record through places, and it reads `.tinderbox/local.json`. It writes nothing.

- A record with a newer version refuses the command. The compare is the full version, so `0.7.1` is newer than `0.7.0`. The next step is to update the CLI.
- A malformed record refuses the command. The next command is `flint tinderbox check`.
- A `.tinderbox` that is a link, or a `local.json` that is newer or that cannot be read, refuses the command.
- A refusal exits 1 with the reason. Nothing is written.

These commands run the gate: `add`, `import`, `remove`, `rename`, `rename --tinderbox`, `dissolve`, `connection add`, `connection remove`, `repo add`, `repo remove`, `heal`, `org set`, `git sync`, `git resume`, `git resolve`, and `git publish`. `heal --dry-run` gives the same refusal as `heal`. `flint tinderbox sync` alone does not refuse. It reports the refusal of the record as an issue, does not write `tinderbox.json`, and is `partial`. `sync --dry-run` asks the same gate: it gives the same issue and proposes no write of `tinderbox.json`.

## Members

A member is a Flint that `[flints]` of `tinderbox.toml` declares. Each entry has a `name` and a `source`. It can also have a `mode` and a `type`. The `name` must be the `[flint].name` of the member.

- **Owned member** (`mode = "own"`; the default for a Git source). The box owns the Flint. The source is a Git source. The local sync clones it into the box as `(<Type>) <name>`, for example `(Flint) NUU Mesh`. Each machine that has the box clones the same member. When the roster lists a Flint with the name of an absent owned member at another path, the local sync does not clone it. The member is `blocked` with the next command `flint tinderbox import <name>`. `git sync` gives the same block.
- **Reference member** (`mode = "reference"`; the default for `registry:<name>`). The Flint stays at its path on this machine. The local sync finds it in the roster (`~/.nuucognition/places.json`) and wires it. It never clones, copies, or moves it. Every reference spelling works: the name, the folder form, and the address (`@org/flint/<slug>`, `@/flint/<slug>`). When the roster of this machine does not have the Flint, the member is `blocked`. The next command is `flint create <name>`, or `flint register <path>` when the Flint is on disk.
- **Required and optional.** `[flints].required` holds the members that the box needs. `[flints].optional` holds the members that some machines cannot reach. A failed clone of an optional member is a warning, not a failure. Its row says `skipped (optional, not on this machine)`. An optional member with a roster collision is `skipped` with a warning.
- **Type.** `type` (a lowercase slug) is the Flint type of the member. It sets the folder prefix: `type = "computer"` gives `(Computer) <name>`. The default is `flint`.

"On this machine" is the column `PRESENT` of `status`. It means an owned member that is cloned in the box, or a reference member at its roster path.

A **Git source** is a source that `git clone` accepts. One classifier decides it for `import`, `start`, `add`, and the validation of `tinderbox.toml`:

- A URL of a Git transport: `https`, `http`, `ssh`, `git`, `file`, `ftp`, `ftps`, `git+ssh`, or `ssh+git`.
- SCP syntax with any user or with no user, for example `alice@host:repo.git`.
- `<helper>::<address>`, or an absolute local path.

A relative path is refused, because `tinderbox.toml` goes to other machines. `import` and `start` resolve a relative origin to its absolute path before the move. `registry:<name>` is the source of a reference member. `path:<dir>` is the source of a reference repo.

## Repos and connections

A **repo** is a codebase that is not a Flint and that the box shares with its members. `[repos].required` declares it.

- An owned repo has a Git source. The local sync clones it into `Repos/<slug>/`. The slug is the last path segment of the source, without `.git`, and it must be one folder name. A source that gives `.`, `..`, `.git`, a path separator, or a folder outside `Repos/` is refused before any write.
- A reference repo has `path:<dir>` as its source. It stays at its path.
- `exposed-to` is `"all"` (the default) or a list of member names. Each of these members gets a codebase reference to the repo in its `flint.toml`.

A **connection** is a Flint reference that the local sync wires from one member to another. `[connections].required` declares it.

- `{ from = "A", to = "B" }` gives A a reference to B.
- `{ interconnected = ["A", "B", "C"] }` gives each member of the group a reference to each other member.
- `kind` (optional, a lowercase slug) names the relation.

The local sync writes each reference into `[references]` of `flint.toml` of the member and fulfils it in `.flint/references.json`. The box records each reference from one member to another that it wired in `wired` of `tinderbox.json`. `sync`, `heal`, `check`, `git sync`, `connection remove`, `remove`, and `rename` act only on these wired references. The local sync removes a wired reference when no connection declares it now. A reference that a person added (for example with `flint reference flint <name>`) stays. `repo remove` and `connection remove` strip the references that they wired. A strip that fails is reported as a failure, and no line says "stripped" for it. A strip that empties a list removes the key.

## Local sync

`flint tinderbox sync` is the local sync of the box. It makes the box match its intent on this machine. It clones a missing owned member, references a roster member, registers the members, wires the connections and the repos, and records the box. The clone is the one fetch: bytes that the intent names and that are absent, the same exception that `flint sync` has. Then it runs the local `flint sync` in each selected member. It never asks the registry, never moves a lock, and never touches the origin of a member. `--dry-run` shows the plan and writes nothing.

The box phase, in order:

1. Read and validate `tinderbox.toml`. A failure stops the run before the first write.
2. Make the box repo when it is absent, and refresh `.gitignore` of the box.
3. Clone each missing owned repo and each missing owned member.
4. Find each reference member in the roster.
5. Register the members in the roster, and as Obsidian vaults unless `--no-open` is set.
6. Wire the repos and the connections into the members, and remove each wired reference that no connection declares now.
7. Record the box in `tinderbox.json`.

Then the member runner runs `flint sync` in each selected member, in this process, 5 at a time. The box phase does not make the standard folders of a member: the `flint sync` of the member makes them.

Rules:

- The one fetch is `git clone` of an owned member or an owned repo that the intent names and that is absent.
- The box phase never moves or deletes a folder. Undeclared folders stay. `check` and a run with no `--only` and no `--skip` report them. The `flint sync` of a member can move or remove folders that Flint owns in that member: a repository snapshot folder, a marker folder, or a shard build that a rename moves.
- The plan of `--dry-run` names the box repo, `.gitignore`, `.tinderbox/`, the roster rows, the clones, the wiring, and `tinderbox.json` (made, upgraded, or written). It names each Obsidian vault that the run registers; with `--no-open` it names none. Then it gives the plan of each selected member.
- In a dry run, an absent owned member has the plan state `planned`. Its row says `would clone, then sync`, and its JSON has `"status": "planned"` and `"plan": "clone"`.
- A newer or malformed record does not stop the local sync. The run reports it as an issue, does not write `tinderbox.json`, and is `partial`. This is the one exception to the write gate.
- `--only <names...>` and `--skip <names...>` select members. The repos are synced only in a run with no `--only` and no `--skip`. A run with `--only` or `--skip` reports no undeclared folder.
- In a box with an org, an owned member with no org or another org is a notice. Its next command is `flint tinderbox org set <org> --id <uuid> --apply`. The local sync never writes the org of a member.
- `--json` prints one operation report for the box, with `members[]` (one report for each member). A refusal is one JSON value too (see "Status, exit codes, and next commands").
- `--no-git` and `--no-update` are retired. Each prints one line that names the right command, and the sync runs.

## Transport

`flint tinderbox git sync` is the transport of the box. It exchanges the history of the box repo and of each member with origin, with the local sync in the middle. The order is the order of `flint git sync`: checkpoint, fetch, integrate, local sync, checkpoint, push. `--no-sync` is transport only. The Git journal and the lock of a run live in `.tinderbox/`. A reference member is `skipped (reference)`: its own Flint moves its history.

The steps of a run:

1. **The box:** checkpoint, fetch, integrate. No push yet. When the box does not integrate, the run stops before the members: a halt exits 2 (`blocked`), and an error exits 1.
2. **The local sync of the box:** the box phase of `flint tinderbox sync`, with no member runner. A member that the pull declared for the first time is cloned here.
3. **The members,** 4 at a time. Each member runs the flow of `flint git sync`: checkpoint, fetch, integrate, the local sync, checkpoint, push.
4. **The box:** the second checkpoint (it commits the writes of the local sync), then the push.

Rules:

- With `--no-sync`, no local sync runs in the box or in the members.
- `--only <names...>` selects members. The box repo always syncs.
- `--force-local` and `--force-remote` choose the strategy for overlapping changes in each member, as in `flint git sync`.
- The write gate runs before any Git work in `git sync`, `git resume`, `git resolve`, and `git publish`. A newer or malformed record, or a `.tinderbox` that is a link, refuses the command with exit 1. Nothing is written.
- A conflict in the box repo aborts the rebase of the box, because no command resolves the box repo. The run is `blocked`. The next step is Git in the box repo: `git pull --rebase`, resolve, `git rebase --continue`, then `flint tinderbox git sync`.
- A required member that is still not on this machine after the local sync of the box is a failed row. It keeps the reason and the next command of that local sync. When the roster lists the Flint at another path, the member is `blocked` with `flint tinderbox import <name>`. With `--no-sync`, an absent member is `failed` with the next command `flint tinderbox sync`. An optional member that is not on this machine is `skipped (optional, not on this machine)`.
- A conflict in a member holds the rebase. The member is `blocked` with one next command: `flint tinderbox git resolve <member>`. The text "the rebase was aborted" prints only when a rebase was aborted.
- A merge, a cherry-pick, or a revert that holds a member blocks the member (exit 2). The run changes nothing in it. The next command is `git <kind> --continue` in the member. Then run `flint tinderbox git resolve <member>`. The same halt in the box repo stops the run before the members, with the next command `git <kind> --continue` in the box repo.
- Each member run holds the Git run lock of the member (`<git dir>/flint-git-sync.lock`), as `flint git sync` does. When another Git run holds it and the wait times out (60 seconds), the member row is `blocked` (exit 2) with the issue `member-locked`.
- The journal `.tinderbox/git-sync-state.json` records the box, each member of a run, and the member filter of the run (`--only`). The lock `.tinderbox/git-sync.lock` stops a second `git sync` of the same box on this machine.
- The run keeps the lock until every started member ends. When a checkpoint of the journal fails, no new member starts, the box does not push, and the journal stays. The run exits 1 with the next command `flint tinderbox git sync`.
- `git status` shows the journal and the Git state of each member. It writes nothing. A reference member is never `pending`. `git status` reads no journal through a `.tinderbox` link: it names the link as a failure.
- `git resume` continues the last run after an interruption. It does the box first: checkpoint, fetch, integrate, and the local sync of the box. Then it selects the members again. It takes each member with an open entry and each member that holds a rebase. When the box did not finish, it also takes each member of the filter of the run with no entry. So a member that the pull declared is transported in the same run.
- In `git resume`, an absent member is a failed row with its next command. The journal stays until every required member is settled. A journal entry of a member that `tinderbox.toml` no longer declares is settled as skipped, so the journal can end. `Nothing to resume` prints only when no entry is open and no member holds a rebase. `git resume` never advises `resume` again.
- `git resolve <member>` fixes one member. It continues the held rebase, or it syncs the member again with `--local` or `--remote`. Use `--replay` after a remote history rewrite. It takes the Git run lock of the member before it reads or aborts the held rebase.
- `git publish <url>` sets origin of the box repo, names the branch `main`, commits pending changes, and pushes. It compares the configured URL of origin (`git config --get remote.origin.url`), so an `insteadOf` rule is not a change of origin. It asks unless `--yes`. A declined prompt exits 1, and the next command adds `--yes`. With `--json` or with no terminal, it refuses unless `--yes` (exit 1), and nothing is written.
- Each Git command takes `--json`: one JSON value on every path. The block of members is `Members` and the row of the box is `Box` in each Git command.

## Status, exit codes, and next commands

Each Tinderbox command renders as `flint sync` and `flint git sync` do. It prints one title line with the status, sections with glyphs, and each block once. `dissolve` and `org set` use the same report.

`tinderbox sync` and `tinderbox git sync` use one status rule. The rows of the members and the steps of the box give the counts, and the counts give the status:

| Status | Meaning | Exit |
|---|---|---|
| `ok` | Each member is done, current, or skipped | 0 |
| `partial` | Some members failed, and other members succeeded | 1 |
| `failed` | Every member failed, or the command was refused | 1 |
| `blocked` | A block and no failure. Examples: a held rebase, a held merge, cherry-pick, or revert, a lock wait that timed out, an absent reference member, a roster collision, a halt of the box repo | 2 |

- A step of the box counts only when it changed something, failed, or is blocked. A required member that is still not on this machine is a failed row, not a fatal issue.
- A halt of the box repo is `blocked` and stops the run before the members.
- A skipped row gives its reason in brackets in both verbs, for example `skipped (reference)`.
- `tinderbox sync` exits as `flint sync` does. `tinderbox git sync`, `git resume`, and `git resolve` exit as `flint git sync` does.
- A refusal (no box, an invalid `tinderbox.toml`, a refusal of the write gate, a bad argument) exits 1 with the reason and one next command on a `Next` line. The reason does not repeat the next command.
- `check` counts each finding as failed and exits 1 when it has a finding. A warning and a notice do not count. A required reference member that the roster does not have is a finding. A malformed or newer `tinderbox.json` is a finding. A member with its declared name in a folder with another type prefix is the finding `folder`. `heal` moves that folder and does not write `tinderbox.toml`.
- `status` exits 0 also when this CLI cannot read `tinderbox.json`. Its title then says `org not known (tinderbox.json cannot be read)`, and a warning gives the next step.
- A dry run marks nothing `synced` and counts nothing `done`. A blocked member in a dry run is `blocked`, not `failed`. An absent owned member in a dry run is `planned`.
- Each issue gives at most one next command, and the report prints each next command once.
- `--json` prints one JSON value on every path, with `status` and `next`. `status --json` adds `org`, `synced`, `members`, and `repos`.
- In `--json`, a refusal is a report with `status: "failed"` and one fatal issue with `code: "refused"`. `issues[].target` names what the refusal is about: a file, a member, or a folder. `issues[].next` is the next command, as in the top-level `next`. `issues[].data.key` is the key of `tinderbox.toml` for a validation failure, for example `flints.required[0].source`. After a failed operation, `issues[].data.restore` says whether every completed step was undone.
- A wait for a lock that timed out is a report with `status: "blocked"` and one issue with `code: "lock-timeout"` (exit 2).

## The commands

Every command runs from the box root or from any folder in a member (see "Where to run"). A command that writes nothing is marked "reads".

| Command | What it does | Flags | Exit |
|---|---|---|---|
| `init <name>` | Makes `(Tinderbox) <name>` in the current folder with the three files and a first commit. | `--no-open` (it has an effect only with `--from`) | 0, 1 |
| `init --from <url>` | Clones the box from its Git URL into the current folder, then runs the local sync. It prints one report. A blocked member exits 2. | `--no-open` | 0, 1, 2 |
| `start <name> [path]` | Makes a new box and moves the current Flint into it as its first owned member. | — | 0, 1 |
| `import <name> [source]` | Moves a Flint of the roster into this box as an owned member, or declares it from `[source]` when the roster does not have it. One operation with one restore. | `--no-open`, `--yes` | 0, 1 |
| `add <name> <source>` | Declares a member without cloning or moving it: an owned member from a Git source, or a reference member from `registry:<name>`. It writes `tinderbox.toml` and `.gitignore`, and it records a member with a known id in `tinderbox.json`. It makes `.tinderbox/` and the roster row of the box when they are absent. | `--json` | 0, 1 |
| `remove <name>` | Removes a member from `tinderbox.toml` and the record, and strips the wired references of the member. A reference that a person added stays. The folder and its roster row stay, except with `--move-out`: the folder then moves into `<dir>`, and the roster row follows it. One operation with one restore. | `--move-out <dir>`, `--json` | 0, 1 |
| `rename <from> <to>` | Renames a member: its declaration, its record entry, its `flint.toml` name, its folder, its roster row, and its Obsidian vault. It rewrites the wired references to the new name. A reference that a person added keeps every byte. It backs up both box files first. One operation with one restore. | `--json` | 0, 1 |
| `rename --tinderbox <name>` | Renames the box, its folder, and its roster row, and points the Obsidian vault of each member at the new folder. When only the folder has another name, it moves the folder. It backs up both box files first. One operation with one restore. | `--json` | 0, 1 |
| `dissolve` | Backs up and removes `tinderbox.toml` and `tinderbox.json`, removes the roster row, and strips the wiring. `.git`, the member folders, and `.tinderbox/` stay. One operation with one restore. A member with unsaved work blocks it (exit 2) unless `--force`. | `--dry-run`, `--move-to <dir>`, `--force`, `--yes`, `--json` | 0, 1, 2 |
| `sync` | Makes the box match its intent on this machine, then runs `flint sync` in each selected member. | `--dry-run`, `--json`, `--only <names...>`, `--skip <names...>`, `--no-open` | 0, 1, 2 |
| `status` | Reads: the box (its name, its org, its last sync) and the state of each member and connection. | `--json`, `--wide` | 0, 1 |
| `check` | Reads: compares the intent with this machine and gives one next command for each finding. | `--json` | 0, 1 |
| `heal` | Repairs the drift that `check` finds (with a backup of both box files), then runs the local sync of the box. It exits 1 when a finding stays. | `--dry-run`, `--yes`, `--json`, `--no-open` | 0, 1 |
| `repo add <name> <source>` | Declares a repo and clones it from a Git source into `Repos/`, or references it at `path:<dir>`. | `--exposed-to <all\|names>`, `--mode <own\|reference>`, `--json` | 0, 1 |
| `repo remove <name>` | Removes a repo and strips its codebase reference from each member. | `--purge`, `--yes`, `--json` | 0, 1 |
| `repo list` | Reads: the repos, their mode, source, path, members, and whether each is on this machine. | `--wide` | 0, 1 |
| `connection add [from] [to]` | Declares a connection: one direction, or an interconnected group. | `--group <names...>`, `--kind <slug>`, `--json` | 0, 1 |
| `connection remove [from] [to]` | Removes a connection and strips the references that it wired. | `--group <names...>`, `--json` | 0, 1 |
| `connection list` | Reads: the connections and whether the local sync wired each one. The alias of `connection` is `connections`. | `--wide` | 0, 1 |
| `git sync` | Exchanges the history of the box repo and of each member with origin, with the local sync in the middle. | `--no-sync`, `--only <names...>`, `--force-local`, `--force-remote`, `--json` | 0, 1, 2 |
| `git status` | Reads: the Git journal of the last or interrupted run and the Git state of each member. | `--json` | 0, 1 |
| `git resume` | Continues each member that the last `git sync` did not finish. | `--no-sync`, `--json` | 0, 1, 2 |
| `git resolve <member>` | Fixes one blocked member: continues its held rebase, or syncs it again with a strategy. | `--local`, `--remote`, `--replay`, `--no-sync`, `--json` | 0, 1, 2 |
| `git publish <url>` | Sets origin of the box repo, names the branch `main`, commits pending changes, and pushes. With `--json` or with no terminal, it needs `--yes`. | `--yes`, `--json` | 0, 1 |
| `org set <org>` | Prints the plan to set the org of the box and of each owned member, and writes it with `--apply`. | `--id <uuid>`, `--apply`, `--json` | 0, 1 |

## Safety rules

- **The commands that move a folder:**
  - `start` moves the current Flint into the new box.
  - `import` moves a Flint of the roster into the box. It asks unless `--yes`.
  - `remove --move-out <dir>` and `dissolve --move-to <dir>` move member folders out of the box.
  - `rename` moves the member folder and points its Obsidian vault at the new folder. `rename --tinderbox` moves the box folder and points the vault of each member at the new folder.
  - `heal` gives a member folder its canonical name. It moves a broken member folder aside as `<folder>.broken-<epoch milliseconds>`.
  - `flint move` of a member asks unless `--yes`. It removes the member from `tinderbox.toml`, then moves the folder.
- **The one command that deletes:** `repo remove --purge` deletes the clone under `Repos/`. It asks unless `--yes`.
- **The box phase of `flint tinderbox sync` never moves or deletes a folder.** It clones what is absent and keeps undeclared folders. The `flint sync` of each member can move or remove folders that Flint owns in that member.
- **The commands that write `tinderbox.toml`:** `init`, `start`, `import`, `add`, `remove`, `rename`, and `heal`. Also `repo add`, `repo remove`, `connection add`, `connection remove`, and `flint move` of a member. `rename` and `heal` back up `tinderbox.toml` and `tinderbox.json` to `.tinderbox/backups/` first. `dissolve` backs up both files first.
- **The write gate comes first.** A write command refuses a newer or malformed record before its first effect, and nothing is written. See "The write gate".
- **One operation, one restore.** `import`, `add`, `remove`, `rename`, `rename --tinderbox`, and `dissolve` plan every effect first. They keep the old bytes and roster rows. Then they do the effects, and the change of `tinderbox.toml` comes after the moves and the strips. When a step fails, the command undoes every completed step and exits 1. When an undo fails, the message names it, and the next command is `flint tinderbox check`. `dissolve` reports the box as dissolved only after both files and the roster row are removed.
- **Preview first.** Run `--dry-run` first where it exists: `sync`, `heal`, `dissolve`. `org set` without `--apply` is a plan. `heal --dry-run` names each repair and each effect of its local sync: the record upgrade, `.tinderbox/`, the roster rows, and the Obsidian vaults. For a member that `heal` renames, it also names the folder move, the roster row, the vault that `heal` points at the new folder, and each wired reference to the old name that `heal` removes. `heal --no-open` registers no new vault. It still points the vault of a moved member at the new folder, so no vault names a folder that is gone.
- **The commands that write nothing:** `status`, `check`, `git status`, `repo list`, `connection list`, each `--dry-run`, and `org set` without `--apply`.
- **`dissolve` protects work.** It refuses when a member has uncommitted or unpushed work, unless `--force`. With no terminal to ask, it refuses unless `--yes`.
- **Know which box you are in.** A command walks up to the nearest `tinderbox.toml`. A command in any folder under a box acts on that box, also in a repo under `Repos/`. Never run a write command in a box that you do not own. Scripts and checks run in a scratch box with an isolated home.
- **Use the commands, not the file system.** Do not move, rename, or delete a member folder by hand. Use `remove`, `rename`, or `dissolve`.

## A local box (no org)

- A box with no org is local. It reads clean: no command asks for an org or an account. Its address is `@/tinderbox/<slug>`.
- `flint tinderbox org set <org> --apply` gives the box and each owned member the org. `--id <uuid>` gives the id when this machine does not know the org yet. Without `--apply` the command prints the plan and changes nothing. A reference member keeps its own org.
- `flint tinderbox org set none --apply` removes the org of the box and of each owned member. `none` takes no `--id`.
- In a box with an org, an owned member with no org or another org is a notice. `sync`, `sync --dry-run`, and `check` give it, with the next command `flint tinderbox org set <org> --id <uuid> --apply`. No command heals it: the local sync never writes the org of a member.
- `status` shows the org of the box in its title, or `no org (local tier)`. A `tinderbox.json` that this CLI cannot read is not local: the title says `org not known (tinderbox.json cannot be read)`.
- A `registry:` source resolves from the roster with every reference spelling, the `@/` address included.
- No Tinderbox command checks the account.

## Where to run

- Each command except `init` and `start` walks up from the current folder to the nearest `tinderbox.toml`. So it runs from the box root or from any folder in a member. The walk stops at the home folder and at a file-system boundary.
- With no `tinderbox.toml` above the current folder, a command refuses with one text and the next command.
- A folder that holds both `tinderbox.toml` and `flint.toml` is refused. The refusal names `flint.toml`.
- `init <name>` makes the box in the current folder, and `init --from <url>` clones it there. Both refuse in a Flint or in a folder of a Flint (the next command is `flint tinderbox start`), and in a box.
- `start` runs in the Flint that becomes the first member. The Flint must have a Git origin that is a Git source. A relative local origin is resolved to its absolute path.

## See also

- [[dev-knw-f-cli]] — the command table of the Tinderbox, with all other Flint commands.
- [[(Spec) Tinderbox System]] — the spec: Configuration, Discovery, Sync Protocol, Health, CLI, Move.
- [[Guide - Using Tinderbox]] — the guide for the operator.
- [[(Spec) Flint State Contract]] — what `flint.toml`, `flint.json`, and `.flint/` hold.
