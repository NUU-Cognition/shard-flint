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
  - "[[0e5e27b5-779f-4068-941e-a93a7a39d3c8]]"
  - "[[1fe72314-c531-4439-81fd-b8f136b7e2cc]]"
  - "[[65e535a9-3f77-404f-80b7-32bd43e9ca77]]"
  - "[[a6b4b0bd-1465-4645-b800-e12693e58dfa]]"
  - "[[b61b6398-0bf0-4b33-81ef-df08f56f2fc5]]"
  - "[[ffb27033-03e2-4fb2-850c-a9f83a880880]]"
  - "[[5e7386c8-d7ac-4fbb-886d-6f9e5a41023b]]"
  - "[[d0c32612-4481-4a28-b4d5-75859b6e590e]]"
  - "[[ec13c5bf-9eab-4982-b35a-cffc13d48950]]"
---

# Knowledge: CLI Reference

Quick reference for Flint CLI commands relevant to agent workflows.

## Artifact Helpers

```bash
flint helper artifact create <Type> --title "<Title>"   # Reserve the next number and write the file
flint helper artifact create Task --title "Fix login"   # → Created Mesh/Types/Tasks/(Task) 119 Fix login.md
flint helper artifact create Task --title "Fix login" --content-file draft.md   # Write the full content
flint helper type newnumber <Type>    # Read the next number (zero-padded, 3 digits); it reserves nothing
```

To make a numbered artifact, use `flint helper artifact create`. It holds a lock on the type while it takes the number and writes the file, so two agents never get the same number. It writes `(<Type>) <NNN> <Title>.md` in the folder of the type: the folder of the artifacts of that type, else the location of the type definition. When more than one folder fits, it refuses; then set `--folder <path>` (a folder under `Mesh/`). With no `--content-file`, the file has only the frontmatter `id`, `tags`, and `authors`; then edit the file. A `--content-file` can contain the tokens `{{number}}`, `{{id}}`, `{{name}}`, and `{{author}}`; the command replaces them. `--json` prints the result as one JSON value.

`flint helper type newnumber` is a read. It shows the next number and reserves nothing. Do not write a file with the number that it gives: another agent can take the same number.

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

## Setup, Identity, and New Flints

`flint setup` sets up this machine in seven steps. Each step has a header (`Step 2 of 7 - Git author`), and a summary block ends the run. Setup needs internet access. It checks the network before the first step: with no network it stops with `Flint setup needs internet access. Connect to the internet, then run flint setup again.` (exit 1, code `setup-no-network`), asks nothing, and writes nothing.

1. **Identity**: your Name and the machine display name (the name that people read, such as "Ada Laptop"). The machine name (`machine-slug`, the short word after `#` in an address) comes from the machine display name. The branch prefix (`machine-name`) takes its default from the hostname. They are in the NUU config (`config.toml` in the NUU home, `~/.nuucognition` or `$NUU_HOME`); every NUU CLI reads them.
2. **Git author**: shows the Git author, and asks the email when Git has none. A new Flint needs it for its first commit.
3. **Environment**: checks Git, Node.js, Obsidian, and the CLI of each target of the Orbh default target list (`claude`, `codex`). A missing tool is `✘` with its install command. It never stops the setup.
4. **Agent default**: first, when the Orbh home has no `default.json`, copies the Orbh profiles from the public repository `NUU-Cognition/orbh-profiles` (the pull of `flint orbh profiles update`, under the loading row `Copying the Orbh profiles`; it never writes `profiles.json`). It refuses a `default.json` of the repository that is empty, not valid, or has no profile, and a `default.json` in the Orbh home with no profile does not count as a copy. When the network goes after the check before step 1, or the pull fails, setup warns, names `Later: flint orbh profiles update`, and continues. Then it shows the default target list and the target that Orbh selects now. In a terminal, the person can put another target first; setup writes that order to the machine layer (`defaultTargets` in `~/.nuucognition/orbh/settings.json`).
5. **Computer Flint**: always makes the Computer Flint when it is missing, with no question. A Computer Flint that setup cannot make fails the run (exit 1, code `computer-not-made`, with the repair command). `--computer` and `--no-computer` are retired: each prints one line, and the run continues.
6. **Guide**: clones the Guide of NUU Cognition (`https://github.com/NUU-Cognition/guide-nuu-cognition`) into `(Guide) NUU Cognition` in the home folder, beside the Computer Flint (the answer "yes" is the default). The Guide is a public Flint that a person asks questions with an agent; its Flint Help shard answers Flint questions and explains Flint problems. When the network goes after the check before step 1, or the clone fails, setup warns, names `Later: flint setup guide`, and continues. `flint setup guide` clones it alone.
7. **Open and onboarding**: opens the Computer Flint in Obsidian on every run, and starts the onboarding session there (the launcher of `flint setup onboard`, below). The first open of a machine has the one trust click ("Trust author and enable plugins"); the plugin then reloads Obsidian once, and the session starts in a terminal tab after that reload. A later run resumes the recorded onboarding session. With `--no-open`, with Obsidian off, or with no Obsidian, the session runs in this terminal after the summary. With `NUU_HOME` set, the step opens nothing.

With `--yes`, setup asks nothing and clones the Guide. With `--json` or no terminal, it asks nothing, takes the flags, and starts no session: step 7 opens the Computer Flint and names `flint setup onboard`. With no `--guide` and no `--yes`, it skips step 6. A second run confirms the stored values, keeps the Computer Flint and the Guide, opens the Computer Flint, and resumes the onboarding session. Step 1, step 5, and a stop (Ctrl-C) fail the run; each other step gives a warning and a `Later:` line.

`flint setup onboard` starts or resumes the onboarding session in the Computer Flint. The prompt (`packages/flint/prompts/onboarding/welcome.md`) explains Flint and helps the person make the first Flint. The route, in this order: `obsidian-tab` (the plugin of the Computer Flint runs: the tab starts now), `waits-for-trust` or `waits-for-obsidian` (the CLI writes `.flint/onboarding/request.json`, then opens the Computer Flint; the plugin starts the tab), `same-terminal` (a terminal only), `new-window` (no terminal), and `printed`. The CLI records the session in `.flint/onboarding/session.json`. `--json` gives `{ ok, route, sessionId, resumed, computerFlint, handoffNote?, reason, next? }` and never uses the terminal. `--handoff <file>` writes the handoff note of the setup agent (`Mesh/(System) Setup Handoff.md`) and starts a new session that reads it first.

`flint config org set <slug>` sets the org of this machine: each new Flint gets it. The id comes from `--id <uuid>`, from the enrollment of this machine, or from a Flint of that org on this machine. `flint config org unset` makes new Flints local again. A NUU account and an org are optional.

```bash
flint setup                              # The seven steps (once per machine; a second run confirms)
flint setup --yes                        # Ask nothing: take each default and clone the Guide
flint setup --no-open                    # Do not open Obsidian: the onboarding session runs in this terminal
flint setup --no-guide                   # Skip step 6 (later: flint setup guide)
flint setup guide                        # Clone the Guide of NUU Cognition alone (a present Guide stays)
flint setup onboard                      # Start or resume the onboarding session in the Computer Flint
flint setup onboard --handoff <file> --target claude --json   # The form of the agent of the setup page (no terminal)
flint setup onboard --print              # The plan: the target, new or resume, the session id; writes nothing
flint setup --name "<Name>" --machine-proper-name "<name>"   # Give the answers of step 1 as flags
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

`flint create` prints `Or start an agent: cd '<path>' && flint orbh i`. After the trust click of step 6, a new Flint opens in Obsidian with its plugins on and no trust prompt.

## The Computer Flint

The Computer Flint is the home of this machine: a Flint of the type `computer`, one for each machine. Its folder is `(Computer) <Machine Display Name>` in the home folder, for example `~/(Computer) Ada Laptop`. The word `computer` is reserved for it: `flint open computer` and `flint cd computer` resolve to it from any folder, in any letter case, before any other name. The old spelling `your computer` resolves to it too. No other Flint can take the word: `flint create`, `flint rename`, the registry, and the machine display name refuse it, a Flint that arrives with the name `Computer` gets the local name `Computer (1)` with a warning, and `flint doctor` names each such Flint with a rename. With no Computer Flint, the word refuses with `Next: flint setup computer`. It is private, and it is a local Git repository with no remote (the next `flint setup` or `flint setup computer` makes a missing repository). It holds `Mesh/Start Here.md` (what it is for, how to make a Flint, start an agent, and use `flint doctor` and `flint fix`), `Mesh/(System) Computer Flint.md`, and the repair notes of `flint fix`. `flint open` opens a new Flint through it with trust when no other vault runs the NUU Flint plugin.

The machine display name and the name of the Computer Flint are one name. Two commands rename both: the folder, the `name` in `flint.toml`, the roster row, the vault registry entry of Obsidian, and `machine-display-name` in the NUU config. The word `computer` follows. The machine name (`machine-slug`) does not change; the last line says `The machine word of addresses stays <slug>.` A name of another Flint of this machine, or an open vault, refuses before any write.

```bash
flint setup computer                     # Make the Computer Flint later (--dry-run); it never converts a Flint
flint setup computer --show-claim        # The claim of this machine (~/.nuucognition/computer-flint.json): held or stale
flint rename "<name>"                    # In the Computer Flint: rename the machine and the Computer Flint
flint setup --machine-proper-name "<name>"   # The same rename, from any folder
flint move computer "<new path>"        # Move it; the claim follows, so Flint still finds it
```

`nuu setup --machine-proper-name` renames only the machine and prints `Later: flint setup --machine-proper-name '<name>'`. Do not move the Computer Flint by hand: the claim then names a missing folder (`--show-claim` says `stale`).

## Doctor and Fix

`flint doctor` checks this machine and the current Flint and starts no agent. The rows of the machine: Identity, NUU home, Git, Node.js, Obsidian, the terminal of Obsidian (PATH, font, python3), Agent (the default target list of Orbh), Launcher, Computer Flint, Flint registry, Flint version (it names a newer release from the release check and never fails for it), Orbh profiles (it fails with `Next: flint orbh profiles update` when the Orbh home has no `default.json`). In a Flint (the working folder, or `-p <dir>`): Lock, Sync plan, Git, Obsidian payload. Each failed row has one `Next:` command that runs from the working folder. A row whose repair needs judgement has `Next: flint fix`; then the doctor writes one failure record and ends with `Or: flint fix`. It writes nothing else. Exit 0 when each check passes, else 1. `--json` gives `{ ok, kind: "doctor", code, reason, next, flint, checks: [{ id, label, ok, detail, scope, next? }], failureRecord? }`.

When `flint sync`, `flint create` (`init`), `flint git sync`, `flint shard install`, `flint setup` (and its subcommands), or `flint tinderbox sync` fails, Flint writes one **failure record** and prints `Or: flint fix` after the `Next:` lines. With `--json`, the envelope has `failureRecord` (the path) and no line. An input refusal (a wrong command line: an unknown flag, a bad value, a name in use, not in a Flint, a missing Name or Git author) and a stop (Ctrl-C) write no record. The record is `<NUU home>/flint/failures/<id>.json` (mode 600, spec `flint-failure/0.1`) with `command`, `args`, `exitCode`, `code`, `reason`, `next`, `stderrTail`, `stdoutTail` (40 lines each), `flintVersion`, `flintPath`, `cwd`, and `time`. Flint removes the secrets first (credentials in URLs, token forms, secret `name=value` pairs, secret flags) and keeps the newest 20 records.

`flint fix` starts a **repair session**: `flint orbh i --prompt-file <id>.prompt.md -p <Flint>` on the default target. The prompt file is beside the record. The session runs in the Computer Flint; with no Computer Flint, in the failed Flint (and it prints `Later: flint setup computer`); with neither, in the Flint of the working folder. A Flint that cannot select an agent passes the session to the next one (a `Fallback:` row); when none can, the list of this machine gives the target (a `Target:` row). The agent follows [[dev-sk-f-repair]]: it asks the person before each change to the Git history, the lock, or `flint.toml`, and it writes one repair note in the Computer Flint.

`flint feedback ["<text>"]` sends feedback about Flint to NUU Cognition (`--bug`, `--idea`, `--question`; default `feedback`). It builds a report with the text and a context block (the Flint version, the OS and the arch, the Node.js version). `--with-failure` adds only the facts of the newest failure record: the command words, the exit code, the error code, the version, and the time (no arguments, output, reason, or paths). Each line of the text with a secret key, a secret flag, or an `Authorization` or `Cookie` header loses the rest of the line; private key blocks, URL user information, known tokens, and home folder paths go too. It shows the report, asks in a terminal, and then opens a prefilled new issue of `NUU-Cognition/flint-public` (`--email`: an email to `hello@nuucognition.com`); the person submits it. `--no-open`, `--json`, and no terminal only print the report and the link; `--yes` opens with no question. To help a person, an agent runs it with `--json`, shows the report, and runs it again with `--yes` only after the person agrees. A local copy goes to `<NUU home>/flint/feedback/`.

```bash
flint doctor [-p <dir>] [--json]         # Check this machine and a Flint; one next command per failed check
flint fix                                # Repair the newest failure record in a repair session
flint fix --list [--json]                # The failure records, the newest first
flint fix <n|id|file>                    # Repair an older record (n = the number of --list)
flint feedback --bug "<text>" --json     # Build a feedback report and its link; opens nothing without --yes
```

## Updates and the Release Check

`flint update` installs the newest release of the npm dist-tag `latest` of `@nuucognition/flint-cli` (an exact version, with `npm install -g`). It compares the versions by semver, so it never offers an older version. A flint that is a build of a checkout (ndv, a dev build) installs nothing: update the checkout, then build it. `--check` installs nothing; `--json` prints one value with `current`, `latest`, `state` (`newer`, `current`, `ahead`, `unknown`), `available`, and `install` (the channel of `flint where`). The registry is `npm_config_registry` when set.

**The release check.** Once a day, a command in a terminal starts one background process (`flint update --check --background`). It asks the registry for the newest Flint and compares the `default.json` of the Orbh profiles repository with the local file. It writes a state file in the cache folder of the system (`~/Library/Caches/nuucognition/flint/` on macOS, `$XDG_CACHE_HOME` or `~/.cache` on Linux), never in the NUU home. A later command in a terminal prints one line for each newer item, at most once a day: `Flint <version> is available (this machine has <version>). Next: flint update`, and `Newer Orbh profiles are available. Next: flint orbh profiles update`. The command never waits for the network. Nothing installs by itself.

No check runs and no notice prints in a pipe, with `--json`, in CI, in an Orbh session (`ORBH_SESSION_ID`), for `flint update` and `flint setup`, on a machine that is not set up, with `FLINT_NO_UPDATE_CHECK=1`, or after `flint config set update-check off`.

```bash
flint update                             # Install the newest release (an npm install only)
flint update --check [--json]            # Name the newest release; install nothing
flint config set update-check off        # Turn the daily release check off (on turns it back on)
```

## Shard Discovery

A shard is a package with two entities: the **source** (the files a person edits, in `Shards/(Source Local) <Name>/` or `Shards/(Source Remote) <Name>/`) and the **shard** (the build in `Shards/<Name>/`, or the alias as a Title when the alias is not the slug of the name). `flint.toml` is the intent, `flint.json#shards[<shard id>]` is the lock, and the shard registry gives the versions of each package. The words are in the glossary of the spec ([[(Spec) Flint Shards#Glossary]]).

A `<ref>` names one shard of this Flint. One resolver reads it in this order: the alias (its key in `flint.toml`), the shorthand, the address (`@org/<slug>` or `@org/shard/<slug>`), the id (`<uuid>`, `@<uuid>`, or a prefix of eight or more characters), then a former address (`@org/<former slug>` or `@org/shard/<former slug>`) or a former shorthand. A former form prints `moved: <old> is now <new>` and goes on. A bare word is an alias or a shorthand; a name is an address. The name, a former name, a bare former slug, and a folder name are not a ref: use the alias, or the address for a former name. A ref that names two shards is refused as `ambiguous`, with one next command per shard that names its alias.

```bash
flint shard list [--json]             # One row per shard: ID PACKAGE SHORTHAND VERSION STATE (ALIAS when an alias differs from its slug)
flint shard status <ref> [--json]     # The row, the Git state of the source, dependencies, pending migrations
flint shard status <ref> --health     # Also run the health check (it writes nothing)
flint shard info <ref>                # An alias of status
flint resolve <spec>                  # The answer of the walk: this Flint, this machine, or the shard registry
```

```
$ flint shard list
Shards
  ID                PACKAGE                      SHORTHAND   VERSION     STATE
  4426ad5b          @nuucognition/meeting-notes  meet        0.1.0       edited

  1 shard, 1 declared
```

`status` prints one row with the labels `Id`, `Package`, `Address`, `Alias`, `Shorthand`, `Name`, `Version`, `State`, `Request`, `From`, `Registry`, `Folders`, in this order. `Version` is the version of the build here; when the source moved past its build, the row says `stale`.

`--json`: `list` gives `{ rows: <row>[] }`; `status` gives `{ ...<row>, details, moved?, health? }`. A row is `{ id, held, alias, shorthand, name, package, address, request, state, version, from, registry, use, folders: { shard?, source? }, setup, pending, stale }`.

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

**Core shards** are Flint (`@nuucognition/shard/flint`) and Orbh (`@nuucognition/shard/orbh`). Every new Flint gets them, whatever its preset declares; a clone keeps the shard list of its files. When `flint shard list` has no row for one of them, install it:

```bash
flint shard install --core            # Install each missing core shard; leave the present ones alone
```

`--core` installs a missing core shard through the normal install path (its dependencies first, then the lock). It reads the lock by shard id, address, or Git location, never by folder. It takes no other source: with a spec, `--from-git`, `--from-path`, or `--all-dev` it refuses and names the two commands to run. `--json` gives `{ ok, shards, present }`. The core shards install from their Git locations (`NUU-Cognition/shard-flint`, `NUU-Cognition/shard-orbh`), because the shard registry does not serve them as `@nuucognition/<slug>` yet.

## Shard Manifests (loading shards)

`start` assembles a dynamic manifest from each shard's files (init, skills, workflows, templates, knowledge — read from each file's `description` frontmatter). One manifest serves both modes: a headless session runs `start` too, reads the section `# Headless` of the init, and follows the `> [!headless]` callouts of the workflows.

```bash
flint shard start <ref>               # The shard (loads init-<sh>.md), in both modes
flint shard start-dev <ref>           # The source (loads dev-init-<sh>.md), in both modes
```

`flint shard hstart <ref>` and `hstart-dev <ref>` are deprecated (NUU Orbh Task 748). Until the removal wave, they print a notice: with an `hinit-<sh>.md` they load it as before, else they load the init. After the removal wave, they are removed. Use `start`.

The header names the shard: `# Shard: <Name> (<sh>) v<version>`, then `Id`, `Address`, `Alias`, `State` (`published <tag>`, `snapshot <sha>`, or `edited`), and `Request`. A shard bound by reference (`use = "reference"`) loads from the folder in its place. `start` of a shard whose source changed after the build prints `The build of <alias> is stale: its source changed after the build. Next: flint shard build <alias>` on stderr and loads the build.

The start refuses and exits 1 when setup is required (it prints `FORCE SETUP`, the setup file, and the `SETUP REQUIRED` banner), when shard migrations are pending, or when a reference source is gone. With `--json` the output is one JSON value, also on a refusal (`{ ok: false, code, reason, next }`).

## Shard Invocations

An invocation is the durable record of one use of a shard (`.flint/invocations/<id>/events.jsonl`). `start` writes a load record. `invoke <ref> <workflow>` opens a workflow run with its steps, outputs, and parent, and each write ends with the hint lines `now:`, `read:`, `next:`, and `part of:`. In a file of format 0.3, `invoke` first checks that each artifact input is ready (`input-not-ready` gives the ways to make it ready), links to the earlier invocation on the same artifact, and settles the steps that the state, the copy, or the mode settle. A `done` close prints the next workflow. Read [[dev-knw-f-invocations]] before you run a workflow with an invocation file.

```bash
flint shard invoke <ref> <workflow> --input <name>=<value>   # Open a workflow invocation (default parent: the top of the stack)
    [--toward '<shard> <workflow>']                          # The workflow that this one makes the artifact ready for
    [--fresh]                                                # Copy no settled step from an earlier invocation on the artifact
    [--not-ready '<why>']                                    # Open on an artifact that is not ready: only when the person asks
    [--mode interactive|headless]                            # The mode, for a test (default: the mode of the Orbh session)
flint shard step                                             # Where am I: the steps of the top of the stack and the hint
flint shard step done|skip <step>                            # Settle the current step (--evidence '<…>' or --reason '<why>')
flint shard invocations [show|stack|set-input|set-output|close|unlock]   # See, change, and close the records
flint shard invocations --artifact '<artifact>'              # The invocations that took or made an artifact, oldest first
flint shard invocations reindex                              # Rebuild the index of the artifacts (a cache)
flint shard ready '<artifact>'                               # The workflows of the installed shards that take the artifact, with each verdict
flint shard assumed '<artifact>'                             # The assumed states of the artifact and why each holds or not
flint shard assumed set|withdraw '<artifact>' <state> --reason '<why>'   # Set or withdraw an assumed state by hand
```

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
flint shard broadcast <ref>           # Update the shard in each Flint of this machine that installs it from this Flint [--dry-run --only <flint...> --json]
flint shard uninstall <ref>           # Remove the build, its lock record, and its unchanged payload files
```

`flint shard install` with no argument reads every record of `flint.toml`. A record that the lock does not satisfy (no lock line, or the range, the place, or the Git location changed) is installed or built, after its missing dependencies. A record that the lock satisfies is not moved: only `flint shard update` moves a version inside its range. `flint shard update` installs the missing dependencies of the new version first. The pins of the shard repositories (`flint.json#repos`) are lock data: only `flint shard install` with no argument and `flint shard update` write them. Then one registry read per record with a hash records the registry answer and follows a rename that the registry reports. When the registry does not answer, the notice is `The registry did not answer. The answers were not recorded.` `--dry-run` shows the plan and the answers and writes nothing.

`flint shard broadcast <ref>` runs in the Flint that holds the source of a shard, after a change to the source. It updates each direct consumer on this machine: a Flint with a record `@org/<slug>#<this Flint>`, or `#<this machine>` when its walk selects this Flint. A Flint that installs from a consumer is not updated, and a place on another machine is skipped. The broadcast reads the roster one time and stages the package one time. A consumer whose lock holds the hash of the package is `current`, and the broadcast changes none of its files. Each other consumer updates in its own transaction. The broadcast installs no missing dependency, runs no migration, and makes no commit. A consumer that cannot take the package (a range that does not hold its version, a missing dependency, a dependent whose range breaks, a 0.6.0 lock, a newer version in its lock) gets a `refused` or `skipped` row with the reason and the next command to run in that Flint; the other consumers go on, and a `refused` row makes the exit code 1. `--dry-run` prints the rows and writes nothing; `--only <flint...>` limits the consumers.

```
$ flint shard install @nuucognition/meeting-notes@^0.1
✓ Resolved @nuucognition/shard/meeting-notes 0.1.0
✓ Installed Meeting Notes 0.1.0
  Spec    : @nuucognition/meeting-notes@^0.1
  Address : @nuucognition/shard/meeting-notes
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

`flint sync` has two steps: `Resolve Flint` and `Reconcile content`. A Flint with a module record gets a third step, `Reconcile modules` (see § Modules). It never asks the registry, and it never touches the origin of the Flint. It writes no pin, and it never moves a shard of the registry or of Git to another version: it checks out the pin of each shard repository, and it fetches only a pinned commit that is absent. Its writes to the lock: it removes a stale lock line (a line that `flint.toml` does not declare and that has no folder under `Shards/`), it records a rename that the shard reconcile follows, and it records each build that it makes. A build that it restores from the lock keeps the locked version. A build of a source record (`from = "source"`) or of a path record takes the version of that source or folder, and its lock record gets that version. It fetches only bytes that the lock or a declaration names and that are absent or differ: a locked build, the pinned Obsidian payload release when the machine cache does not hold it, a declared source that is not cloned, a repository clone, a source repository snapshot. The step `Reconcile content` installs the pinned Obsidian payload and never moves the pin (see § Obsidian). After the kernel, the row `Fetch workspace repositories` runs `git fetch origin` in each workspace repository (Task 1101): a repository root with an origin that the run did not clone. The fetch moves only the remote-tracking refs: sync never pulls, merges, or checks out. Each repository gets one result against its base (the upstream, else `origin/<default>`): `up-to-date`, `ahead`, `behind`, `diverged` (ahead and behind), `no-base` (fetched, with no base to compare), `skipped`, or `failed` (the fetch failed). The row of each repository carries the result in `data` for `--json`, and the detail of `Reconcile content` sums it (`3 workspace repositories fetched, 2 behind origin, 1 fetch failed`). A repository that is behind, diverged, or failed gives one notice (`workspace-behind` with the next command `git pull --ff-only` in its folder, `workspace-diverged`, `workspace-fetch-failed`). A fetch changes no count and no status. `flint sync --dry-run` and `flint plan` open no socket: they fetch nothing and name the repositories that sync fetches. A halted rebase blocks the run (exit 2) with the next command `flint git sync --continue`. A held merge, cherry-pick, or revert blocks it too: finish it with `git <kind> --continue` or stop it with `git <kind> --abort`, then run `flint sync`. A `flint.toml` that does not parse blocks the run (exit 2): correct the file, then run `flint sync`. When the branch has commits that the cached ref `origin/<branch>` does not have, sync gives the notice `Local history is N commits ahead of origin. Run flint git sync.` A branch with no `origin/<branch>` ref (never pushed) gives no notice. The flags `--no-git` and `--no-update` are retired: each prints one line that names the new command, and the sync runs.

A declared repository, a workspace repository, and a bundle each have a name that is one folder name. Sync checks that each folder is inside its parent folder before it writes. A reference to a Flint that is not on this machine is one report row, not a failure. The next command for a broken codebase reference is `flint fulfill codebase <name> <path>`. `flint reference flint` and `flint reference remove` write the declaration only. `flint reference codebase` writes the declaration and records the path of this machine in `.flint/references.json`. All three print `Next: flint sync`: the sync fulfils the reference.

For shards, sync runs two reconciles. **The shard reconcile** (feature `shards`) makes each build match the lock: it installs a missing locked build from the lock, builds a stale build of a source again, fetches the locked version when the build differs from the lock, follows a rename that the source, the place, or the build shows (`moved: <Old> is now <New> (<address>); the folder, the key, and the type files followed`; the id stays), refreshes the path of a reference, and makes the missing folders of a shard: the spec folders of its build (`skills/`, `scripts/`, `assets/`, …) and the Mesh folders of `folders:` in the manifest. An install makes the spec folders of a build too. A record with no lock line is not current (`not-locked`), with the next command `flint shard install`. A shard repository with no pin is `not-locked` too. A lock from a place heals from that place only. When that place holds a changed source, the record is `not-locked`, with the next command `flint shard update <alias>`. **The source reconcile** (feature `shard-sources`) records the Git state of each source and gives a notice for a source that is a draft, behind, or dirty. It makes the missing spec folders of a source (`skills/`, `scripts/`, `migrations/`, …), because Git keeps no empty folder. It never writes a file into a source. `flint shard clone` makes the same folders after the clone. A missing dependency or a dependency outside its range is not current, with the next command. The rename process is in [[(Spec) Flint Shards . Rename]]. On a Flint with the 0.6.0 shard records, `list`, `status`, and `start` read them with a notice, and every shard write is refused with the next command `flint migrate run`.

For modules, sync runs the same kind of reconcile and then one step. **The module reconcile** (feature `module-builds`, the row `Module builds`) makes each build under `Modules/` match the lock `flint.json#modules`: it restores a locked build that is missing or that differs from the hash of the lock (the locked version stays), builds a record with `from = "source"` again when its source changed, and removes a lock line that `flint.toml` does not record. A record with no lock line is `not-locked`: sync installs no new module, and the next command is `flint module install`. A restore fetches only the Git commit, the source, the path, or the bundle build that the lock names; sync never asks the registry. **The step `Reconcile modules`** runs after `Reconcile content`. It runs the plan of each installed module (one process each), then applies the plan: Flint applies the desired objects (cron schedules, stations) with the managed marker, and the module applies each own action with its verb `apply`. `flint sync --dry-run` and `flint plan` run the plans, show each change as `Would …`, and apply nothing. A module that fails fails its own row only. After a sync that is not a dry run, a running Flint server of the Flint reconciles its live modules. A Flint CLI that runs inside a module process (`FLINT_MODULE_PROTOCOL` is set) runs no module process, so a module cannot run itself again through `flint sync`.

`flint git sync` checkpoints the local work, fetches, integrates, runs the local sync, checkpoints the writes of the sync, and pushes. A conflict halts before the local sync: resolve it, then run `flint git sync --continue`. `flint git merge` does the same with no push. Each next command of a merge uses the verb `merge`: after a conflict, run `flint git merge` again; after a remote history rewrite, run `flint git merge --replay`. `--continue` with `--replay`, `--force-remote`, or `--force-local` is refused. `flint git resolve` takes paths relative to the working directory. A held merge, cherry-pick, or revert refuses `sync` and `merge` with exit 2, and the run changes nothing. The next command is `git <kind> --continue`; then run the command again. One Git run at a time holds the run lock of the repository, `<git dir>/flint-git-sync.lock`: `sync`, `merge`, `--continue`, `--abort`, and `resolve` take it. When the wait for the lock times out (60 seconds), the report is `blocked` with exit 2. When the local sync fails, the push still sends the integrated history as it is, with no second fetch; when origin moved during that sync, the push is rejected, and the next `flint git sync` sends it. A Git halt prints under `Failed` and exits 2. `--json` prints the report as one JSON value.

`flint git init` in an existing repository refreshes the full policy: `.gitignore`, `.gitattributes`, and `rerere.enabled`. The managed block of `.gitignore` (written by `flint git init` and refreshed by `flint sync`) ignores the internals (`.flint/`, `.orb/`, `.orbh/`, …), the env files `flint.env` and `flint.env.local`, the remote shard sources (`Shards/(Source Remote) */`), and, when the Flint has `Modules/` or `[modules]`, the remote module sources and the language build folders: `Modules/(Source Remote) */`, `Modules/**/node_modules/`, `Modules/**/.venv/`, `Modules/**/__pycache__/`. The module builds, the local sources, the settings files, and `flint.env.example` stay tracked. `flint git publish <url>` refuses a folder that is not the root of a repository, with the next command `flint git init`. The passthrough (`flint git <git args...>`) gives the terminal to git, so an editor and `add -p` work. A network command of the passthrough keeps the network timer of the other Git commands.

## Modules

A **module** is a package of the kind `module`. It uses the package core of shards: the spec `@org/name[@range][#place]`, the address `@<org>/module/<slug>` (the short form `@<org>/<slug>` is valid in `[modules]`), versions, a lock, Git sources, and the NUU Shard Registry. A shard serves agents (skills, knowledge, templates, types). A module serves the machine: its code runs as a process (Node.js, Python, or an executable), and `flint sync` makes its objects real on this machine (cron schedules, stations). The CLI bundle ships the first-party modules `crons` and `dms`.

```bash
flint module list [--json]                       # One row for each record of [modules], each legacy table, and each source here with no build
flint module status [<name>] [--json]            # The package state, the sync plan, and the liveness of each module; writes nothing
flint module install <spec> [--name <name>]      # Install a module (@org/name[@range][#place]); then run flint sync
flint module install --from-path <dir>           # Install from a folder of this machine (a source runs its build command first)
flint module install --from-git <owner/repo[#ref]>   # Install from a Git location (runs the build command of the manifest)
flint module install [--dry-run]                 # No input: make the lock and the builds match the records of [modules]
flint module uninstall <name>                    # Remove the build, the lock record, and the record; the settings file and the data folder stay
flint module build <ref> [--broadcast]           # Build from Modules/(Source Local|Remote) <Name>, and record from = "source"
flint module update [<name>] [--dry-run]         # Move each module to the newest version inside the range of its record
flint module release <name> [<version>]          # Release a module source: build, check the committed build output, tag, push, register
flint module start|stop|restart <name>           # A live module under the Flint server of this Flint
flint module control <name> [<action>] [--data <json>]   # List the actions of the controller of a live module, or run one
flint module settings <name> get|set <key> [<value>] [--create]   # Read or write one key of Modules/<Name>.settings.toml; a write keeps every comment
flint module <name> <command> [args...]          # Run a command of a module
```

| Path | Git | Holds |
|------|-----|-------|
| `[modules]` of `flint.toml` | Tracked | One record for each module: `<name> = "<spec>"`, or `<name> = { source = "<spec>", git?, from = "source"?, use? }` |
| `flint.json#modules` | Tracked | The lock: the address, the version, the state, and the hash of each build |
| `.flint/modules.json` | Ignored | The facts of this machine about each module |
| `Modules/<Name>/` | Tracked | The build: `module.yaml` and the runnable files |
| `Modules/(Source Local) <Name>/` | Tracked | A local source |
| `Modules/(Source Remote) <Name>/` | Ignored | A remote source: a clone of a repository, as for shards |
| `Modules/<Name>.settings.toml` | Tracked | The settings of the module in this Flint |
| `.flint/modules/<name>/` | Ignored | The data folder of the module on this machine |

- **The record name** is the key in `[modules]`: the first word of `flint module <name> <command>`, the data folder, and `FLINT_MODULE_NAME`. It is kebab-case. The default is the slug of the module name; `--name` of `install` and `build` sets another name. A reserved verb of `flint module` (`list`, `status`, `install`, `uninstall`, `build`, `update`, `release`, `start`, `stop`, `restart`, `control`, `settings`, `help`) cannot be a record name. The build folder is the module name (`Modules/DMs/` for the record `dms`) when the record name is the slug of that name, else the record name as a title.
- **The walk.** For a module spec, the walk asks this Flint, then the CLI bundle, then the other Flints of this machine, then the registry. So `flint module install @nuucognition/module/dms` installs the build of the CLI bundle. `install` checks the range `flint` of the manifest. A registry version is the released tree with its build output, so its install runs no build step. A Git install and a source run the `build` line of the manifest in the source folder before the copy.
- **Install writes the package only.** `install`, `uninstall`, `build`, and `update` change no object of the machine. Each prints `Next: flint sync`.
- **The settings file** `Modules/<Name>.settings.toml` belongs to the Flint, not to the package. A build, an install, and an update never change it, and `uninstall` keeps it. The key `settings` of the manifest names its keys. A settings file that does not parse fails the plan of its module, and sync does not reconcile that module. With no settings file, the module gets no settings. Flint owns every write of a settings file: `flint module settings <name> set <key> <value>` writes one key (a dotted key, for example `schedules.tick.machine`; the value is one TOML value, so a string needs quotes: `'"check-box"'`; `--create` makes the table of a nested key) under the lock of Flint and keeps every comment, and `get` reads one key. A module that changes its own settings (for example `flint module crons own`) calls this verb. Then run `flint sync`.
- **Desired objects.** A module never writes an object of Orbh. Its plan returns desired objects. Flint stamps each object with the managed marker `{ flintId, module: <record name>, key, declHash }` (a cron schedule also gets `machine`, its owner), creates, updates, or resumes each desired object, and pauses each managed object of the module that the plan no longer desires. Sync never deletes an object of a module, and its history stays. When a record goes away (`flint module uninstall`), the next sync pauses each managed object of that module. A local object (no marker) with the key of a desired object is a report, never a take-over. Each managed object is in the Orb store `.orb/`, which Git ignores.
- **A plan that fails** (a non-zero exit, the time limit of 30 s, or output that does not parse) is the issue `module-plan-failed`. Sync does not reconcile that module, and it never reads a failure as a removal.
- **The lines of a module in the sync report.** Each line of a module in `flint sync` and `flint sync --dry-run` names the module first: `<mark> (module) <name>: <message>`, where `<name>` is the record name in `[modules]`. This holds for a change (`✓`), a planned change (`●`), a change of a desired object (`Created the station demo-desk`), the pause of an object of a removed record (it names the module of the object), and an issue (`!`). A change of a real sync ends with the detail that `apply` of the module returned, after ` — `: `✓ (module) demo: Wrote greeting.json: Hello — wrote .flint/modules/demo/greeting.json`. A dry run has no apply detail.
- **Module logs.** A module can write log lines from `plan` and `apply` (the SDK tool `log`). The section `Module logs` after `Changes` shows them: `· (module) <name>: <message>` for `info` and `⚠ (module) <name>: <message>` for `warn`. A dry run shows the logs of each plan; a real sync shows the logs of each plan and of each apply. Flint shows at most 50 lines of one process and cuts a message after 500 characters, and says so. A log is information only: it never fails a plan, never makes a module not current, and never changes `flint doctor`. `flint module status <name>` shows the lines of the plan with the same prefix, and the logs of the plan in the row `Logs` under the plan. In `--json`, the report has `moduleLogs` (`{ module, source, action?, level, message, cut?, flint? }`; `source` is `plan` or `apply`), and each module change carries `module` and, after a real apply, `detail`.
- **Live modules.** A module with `kind: live` runs a long-running process under the Flint server of the Flint. The server starts it when the server starts, after a `flint sync`, and on `flint module start`. After a sync, the server also stops the process of a module that is no longer installed, and starts again a running module whose build, settings file, or env file changed. A process registers, sends a heartbeat (default every 10 s), and can register a controller for its own actions. After three missed heartbeats the state is `down`. A process that exits starts again after a back-off (1 s, doubling, at most 60 s). `stop` sends SIGTERM, then SIGKILL after 5 s; a stopped module stays stopped until `flint module start` or the next start of the server. `start` and `restart` start the Flint server when none runs; `stop` starts no server. Each process has the Foundation record `flint-module/<name>`, and its output goes to `.flint/run/modules/<name>.log`. `flint module status` shows the liveness when a Flint server of the Flint runs. `flint module control <name>` lists the actions of the controller of a running live module, and `flint module control <name> <action> [--data <json>]` runs one: the server calls the controller with the module token. `--data` is one JSON object (default `{}`). A failed action exits 1. `control` starts no server.
- **Legacy tables.** A table `[modules.<id>]` with no `source` key is a built-in declaration of the 0.7.0 pre-releases. Sync never reads it as a record, and no migration moves it. A legacy table does not make the Flint behind, so `flint sync` runs. `flint module list`, the sync report, and `flint doctor` name it with a step by hand and the next command `flint module install @nuucognition/module/<id>`. To replace `[modules.crons]` or `[modules.dms]` by hand: move its keys, without `enabled`, to `Modules/Crons.settings.toml` or `Modules/DMs.settings.toml` (each `[modules.crons.schedules.<name>]` becomes `[schedules.<name>]`), remove the table, then run `flint module install @nuucognition/module/<id>` and `flint sync`. When the table has `enabled = false`, do not install the module. `flint module install` and `flint module uninstall` refuse a name that has a legacy table.
- **Doctor.** `flint doctor` has the row `Modules`. It passes when each record has a usable build, each plan ran, no issue or report needs a person, and the plan is empty (`dms installed; the plan is empty`). A plan with changes gives `Next: flint sync`. A legacy table is a warning with the step by hand and `Next: flint module install @nuucognition/module/<id>`.
- **Authoring.** Flint runs `node <entry> <verb>`, `python3 <entry> <verb>`, or `<entry> <verb>` in the Flint root, with the verb `plan`, `apply`, `command`, or `live`. Every module process gets the `FLINT_*` values of the module (`FLINT_ROOT`, `FLINT_MODULE_NAME`, `FLINT_MODULE_DATA`, `FLINT_MODULE_SETTINGS_JSON`, `FLINT_CLI`, and more) and every value of the env files. The TypeScript SDK is `@nuucognition/flint-module-sdk` (a module bundles it). The Python SDK is `flint_module.py` (standard library only, Python 3.9 or later); the CLI bundle ships it, and Flint puts its folder on `PYTHONPATH`. `examples/modules/` of the Flint repo has one module of each language. The full contract for an author (the JSON of `plan` and `apply`, the desired objects, the live contract, both SDK APIs), the templates, and the workflow `knap_module` are in the Knap shard: `flint shard start knap`.

## Crons

The module `crons` (`@nuucognition/module/crons`) gives a Flint its own cron schedules. A schedule fires a headless Orbh session with a prompt, on one machine: the **owner machine**. The orchestrator of that machine sweeps the schedules every 60 s. Install it with `flint module install @nuucognition/module/crons`. The schedules are in its settings file, `Modules/Crons.settings.toml`:

```toml
machine = "katana"                       # Optional: the default owner (a machine slug, or a machine id)
target  = "claude/o5h"                   # Optional: the default target; else the Orbh default target

[schedules.morning-brief]
expr        = "0 7 * * 1-5"              # Required: a 5-field cron expression
timezone    = "Australia/Sydney"         # Optional: an IANA timezone; default: the clock of the owner machine
prompt      = "Write the brief of the day."   # Or prompt-file = "Mesh/Agents/Morning Brief.md" (give exactly one)
description = "The brief of the day"     # Optional
# Optional: machine, target, misfire = "coalesce"|"skip", overlap = "skip"|"allow", enabled = false
```

The key of a schedule table is the name of the schedule: lower-case letters, digits, and hyphens. A `prompt-file` is a path relative to the Flint root, inside the Flint. Sync reads the prompt file and stores its text in the schedule, so after an edit of the prompt file, run `flint sync`. To add, change, or remove a schedule, edit the settings file, then run `flint sync`. A removed table or `enabled = false` pauses the schedule; its history stays.

```bash
flint module crons list [--json]         # The schedules of the settings file (owner, state here, next due, LAST: the last fire and its session) and the local schedules; writes nothing
flint module crons own [<name>]          # Make this machine the owner of one schedule (no name: the default owner); writes the settings file
flint module crons run <name>            # Request one fire on the next orchestrator sweep (as flint orbh cron run-now)
flint module crons result <name> [--json]   # The last fire: the session and its state, the result text or the error, the last skip; writes nothing
```

**The owner machine rule.** Only the owner machine fires a schedule. The owner is `machine` of the schedule, else `machine` of the settings file. A value with the UUID shape is compared with the machine id; each other value is compared with the machine slug (`machine-slug` of the NUU config). `flint module crons own` writes the stored machine slug of this machine, because a person reads the slug in a diff, else its machine id. It keeps every comment of the settings file. With no stored slug and no machine id, it refuses with the next command `flint setup`. On the owner machine, sync makes or updates the schedule, and it stores the owner in the managed marker. On each other machine, sync pauses the schedule when it exists there. So when Git brings the settings file to two machines, only the owner machine fires the schedule. Flint does not check that two machines have different slugs, so a slug owner is safe only when each machine that syncs the Flint has a different machine slug. When one exact machine must own a schedule, write its machine id (a UUID) in `machine`: `nuu machine id` prints it. A schedule with no owner fires on no machine: `flint module crons list` and the sync report name `flint module crons own <name>`.

- A schedule that does not parse is an issue, and sync keeps its object as it is. Settings that do not parse (`machine`, `target`, or `schedules` of the wrong type) fail the plan, and sync changes no schedule of the module. An unknown key is an issue, and the module ignores it.
- `flint module crons run` refuses a schedule that this machine does not own, and a schedule that sync has not made yet.
- Each change of the settings file needs `flint sync` on the owner machine. When sync leaves an enabled schedule on this machine and the Orbh orchestrator does not run, sync starts the orchestrator.
- **The fire-time check.** Before each fire, the orchestrator checks three facts with no module process: `flint.toml` still has the record of the module that the marker names (a legacy table is not a record), that module is installed (a build whose manifest parses), and the owner in the marker is this machine. Else it skips the fire with the reason `ineligible`. So after `flint module uninstall crons`, no schedule of the module fires, also before the next sync. A change of the settings file takes effect at the next sync: a schedule that you remove from the settings file fires until `flint sync` pauses it.
- The fired session has no parent. In 0.7.0, its result goes to no person; read it with `flint module crons result <name>`.

**What a cron did.** `flint module crons result <name>` shows the last fire of a schedule: the time and the reason (`due` or `manual`), the session id and its state, the result text of the session or the error of the launch, and the last skip with its reason. A schedule that never fired says so. The next command is `flint orbh result <session id>`. `flint module crons list` shows the same last fire in the column `LAST`. When this machine owns at least one enabled schedule of the Flint and no Orbh orchestrator runs, nothing fires: `flint module crons list`, `flint module crons result`, and the row `Modules` of `flint doctor` then show one warning with the next command `flint orbh orchestrator ensure`. They never start the orchestrator.

**Local schedules stay local.** `flint orbh cron create` makes a local schedule in the Orb store `.orb/`, and Git ignores `.orb/`. So a local schedule does not travel with the Flint, and sync never changes it. `flint module crons list` shows it under "Local schedules". A schedule of a module shows the record name of the module (`crons`) in the column `MANAGED` of `flint orbh cron list`, and `flint orbh cron show` names the module, the key, the owner, and the settings file. `flint orbh cron delete` refuses it: change the settings of the module and run `flint sync`, or remove the module with `flint module uninstall crons`. See [[knw-foh-machinery]] for the cron store.

## Direct Messages

The module `dms` (`@nuucognition/module/dms`) lets a person or an agent send a direct message (DM) to a Flint of this machine. A DM is one text request with one answer. It is an item of the station `dm` of the receiver Flint. It is not a message to a live session. Install it with `flint module install @nuucognition/module/dms`, then run `flint sync`: the sync makes the station `dm`. Its settings file is `Modules/DMs.settings.toml`:

```toml
# Each key is optional: with no settings file, the module serves DMs with the defaults.
target = "claude/o5h"     # The target of the session that answers (<runtime>/<profile>); else the Orbh default target
mode   = "oneshot"        # "oneshot": a session answers the queued DMs, then finishes; "resident": a bound session waits for more DMs
prompt = "..."            # The duty of the session that answers; or prompt-file = "<path>"; else the built-in DM duty
```

The key `allow` accepts only `"local"` in 0.7.0: each process of this OS user can send a DM.

```bash
flint module dms send <flint> "<text>"                          # Send one DM; print the receipt
flint module dms send <flint> "<text>" --wait [--timeout <s>]   # Send one DM and wait for the answer (default 600 s)
flint module dms show <flint> <item>                            # The state of one DM, its text, and its answer; writes nothing
flint module dms inbox [--json]                                 # The DMs of this Flint, newest first; writes nothing
```

- **The sender Flint.** `flint module dms …` runs the module `dms` of the Flint of the working folder, so that Flint must have the module too. Else the command refuses (`This Flint has no module dms.`). To use another Flint, put `-p <dir>` before the module name: `flint module -p <dir> dms send …`.
- **The receiver.** `<flint>` is a name, an id, or an address of a Flint of this machine. A DM to a Flint with no module `dms` fails with the next command `flint module install @nuucognition/module/dms` (run it in the receiver Flint). Before the sync of the receiver, a DM fails with the next command `flint sync` in the receiver Flint. A paused station `dm` refuses a DM too.
- **The receipt.** `send` queues the DM and returns. It prints the receiver Flint (its name and its id), the item id, the sender, and `Next: flint module dms show @<flint id> <item>`. The next command names the Flint by its id, because two Flints of this machine can have one name. With `--wait`, it waits for the answer and prints it. When no answer comes before the timeout, it exits 1, and the DM stays queued. When the receiver fails the DM, it exits 1 with the reason.
- **The answer.** When a DM waits and no session serves the station `dm`, the station starts a session on its target. The session answers each DM with `flint orbh station respond-item dm <item> "<answer>"`. In the mode `oneshot`, the session answers the queued DMs and finishes when no DM is left, so one session can answer more than one DM. In the mode `resident`, one session is bound to the station and waits for more DMs. So a DM can start a real agent session in the receiver Flint. Do not send a test DM to a Flint whose station `dm` has no bound session.
- **The sender** is the Orbh session id, or `@<Name>#<machine-slug>` for a person in a terminal. It is attribution, not authentication.
- **Removal.** `flint module uninstall dms` and `flint sync` pause the station `dm`. No new DM is claimed, and the queued DMs and the answers stay.
- **Local only in 0.7.0.** A DM goes only to a Flint of this machine. A reference that names another machine (`…#<machine>`) is refused. DMs to another machine through the NUU Network are not in 0.7.0.

## Env Files

Three files at the Flint root hold the env values of the Flint:

| File | Git | Holds |
|------|-----|-------|
| `flint.env.example` | Tracked | The names of the env values, with no values |
| `flint.env` | Ignored | The env values of the Flint |
| `flint.env.local` | Ignored | The env values of this machine. A value here wins over `flint.env`. |

- **The syntax** is dotenv: `NAME=VALUE` lines, `#` comment lines, an optional `export ` prefix, single quotes (literal), and double quotes (with the escapes `\n`, `\r`, `\t`, `\"`, and `\\`; such a value can continue on the next lines). An unquoted value ends at a `#` at its start or after a space, and the spaces at each end go. A name has letters, digits, and `_`, and does not start with a digit.
- **Checks.** A name that starts with `FLINT_` is refused (`env-reserved-name`) and is not loaded: only Flint sets a `FLINT_*` value. A line that does not parse is an issue with its line number (`env-parse`), and it is not loaded. When `flint.env.example` exists, the names that no env file sets give one notice (`env-missing`) in `flint module status` and in the sync report. A notice does not change the exit code.
- **Modules.** Each module process gets every value of the env files (`flint.env.local` wins). A module names the values that it needs in `secrets:` of its manifest. A secret that no env file sets is the issue `module-secret-missing` of the module, with the next step.
- **Agents.** An agent can read the Flint-level values from `flint.env.local` and `flint.env` at the Flint root. A value of `flint.env.local` wins. Orbh does not put these values into a session: read a value from the file when the task needs it. Do not write a value into the Mesh, a commit, a message, a session result, or a log. Do not commit `flint.env` or `flint.env.local`. To add a value that the Flint needs, write its name with no value in `flint.env.example`, and its value in `flint.env.local` (this machine) or `flint.env` (the Flint).

## Workspace

```bash
flint workspace list                  # The repositories of this Flint (codebases, URLs)
```

## Tinderbox

A Tinderbox (the box) holds many Flints as one unit. Each Flint of the box is a member.

`tinderbox.toml` is the intent of the box: the name, the members, the repos, the connections. `tinderbox.json` is the record of the box: its id, its type, its version, its org, its members with their ids, and the references that the box wired (`wired`). `.tinderbox/` holds the facts of this machine: the local version, the last sync, the Git journal, the lock, and the backups. No fact lives in two files.

`flint tinderbox sync` is the local sync of the box. It makes the box match its intent on this machine. It clones a missing owned member, references a roster member, registers the members, wires the connections and the repos, and records the box. The clone is the one fetch: bytes that the intent names and that are absent, the same exception that `flint sync` has. Then it runs the local `flint sync` in each selected member. It never asks the registry, never moves a lock, and never touches the origin of a member. `--dry-run` shows the plan and writes nothing. The box removes only the references that it wired, so a reference that a person added stays.

`flint tinderbox git sync` is the transport of the box. It exchanges the history of the box repo and of each member with origin, with the local sync in the middle. The order is the order of `flint git sync`: checkpoint, fetch, integrate, local sync, checkpoint, push. `--no-sync` is transport only. The Git journal and the lock of a run live in `.tinderbox/`. A reference member is `skipped (reference)`: its own Flint moves its history.

Read [[dev-knw-f-tinderbox]] for the model, the write gate, the exit codes, and the safety rules before you run a write command.

```bash
# The box and its members
flint tinderbox init <name>                   # Make (Tinderbox) <name> here with the three files and a first commit
flint tinderbox init --from <url>             # Clone a box from Git, then run the local sync; exit 2 when a member is blocked
flint tinderbox start <name> [path]           # Make a new box and move the current Flint into it as an owned member
flint tinderbox import <name> [source]        # Move a Flint of the roster into the box, or declare it from [source]; --no-open: no Obsidian launch after the move [--no-open --yes]
flint tinderbox add <name> <source>           # Declare a member without cloning or moving it: a Git source (owned) or registry:<name> (reference) [--json]
flint tinderbox remove <name>                 # Remove a member and keep its folder, unless --move-out moves it [--move-out <dir> --json]
flint tinderbox rename <from> <to>            # Rename a member: declaration, record, flint.toml name, folder, roster row, vault registry entry, wired references; backs up both box files first [--json]
flint tinderbox rename --tinderbox <name>     # Rename the box, its folder, its roster row, and the vault registry entry of each member; backs up both box files first [--json]
flint tinderbox dissolve                      # Remove the box and keep the member folders; backs up both box files first [--dry-run --move-to <dir> --force --yes --json]

# The local sync and the health of the box
flint tinderbox sync                          # Make the box match its intent, then run flint sync in each selected member [--dry-run --json --only <names...> --skip <names...>]
flint tinderbox status                        # The box (name, org, last sync) and each member and connection [--json --wide]
flint tinderbox check                         # Compare the intent with this machine; one next command per finding [--json]
flint tinderbox heal                          # Repair what check finds (with a backup of the box files), then run the local sync of the box; exit 1 when a finding stays [--dry-run --yes --json]

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

## Obsidian

The Obsidian payload is the set of released files that Flint installs into `.obsidian/`. The payload source is the repository `NUU-Cognition/flint-dot-obsidian`. The pin (`flint.json#obsidian`) names the payload release of this Flint. The installed state is in `.flint/obsidian/state.json`. Flint owns the release files and verifies them by hash. The person owns the settings files (`app.json`, `hotkeys.json`, `appearance.json`, `plugins/<id>/data.json`).

- `flint sync` installs the pinned payload. It never moves the pin. It fetches the release only when the machine cache does not hold it.
- When the vault is open, `flint sync` does not install. The install waits (a pending install), and the exit code is 0. Close the Flint in Obsidian, then run `flint open "<name>"`: the pending install completes before the launch.
- `flint obsidian update` moves the pin to the recommended release (or to `--to <release>`) and installs it. It keeps every personal setting.
- A release file that changed outside Flint blocks the install (exit 2), and no file changes. Preview the fix with `flint obsidian repair --replace --dry-run`. Do not edit the release files in `.obsidian/`.
- An applied setting is a key of `appearance.json` or `app.json` that the NUU Flint plugin applies at each load. To change one, use `flint obsidian applied set <file>.<key> <value>`, not an edit of the settings file.
- The machine choice turns Obsidian on or off on this machine: `flint config set obsidian off` (or `on`). When it is `off`, Flint does not install the payload and does not open Obsidian.
- With `NUU_HOME` set (isolation), `flint sync` still installs the payload, and Flint does not contact a live Obsidian.
- Community-plugin trust: with the trust setting `auto` (the default), `flint open` asks the NUU Flint plugin of an open vault to open a Flint with trust, after the trust check (the payload is installed, the plugin files are the release files, and `.obsidian/plugins` holds no other plugin). When no vault runs the plugin, the Computer Flint opens first and carries the open (one more window). The plugin never changes a vault that the person put in Restricted Mode in Settings (the trust screen's "Browse vault in Restricted Mode" lasts one load and stores nothing). Obsidian still asks ("Trust author and enable plugins") at the first open of the Computer Flint, with `flint config set obsidian-trust ask`, when the check fails, and with no open vault and no Computer Flint. `flint obsidian status` shows the setting and the check on the line `Trust`. A link opens only a folder in Obsidian's vault registry: with Obsidian stopped, `flint open` registers a new Flint first; with Obsidian running and no entry, it opens nothing and names the manual step (Open folder as vault).
- A 0.6.x Flint can hold a legacy Git payload in `.obsidian/`. Close its vault, then run `flint migrate run`: the step `l5` converts it and saves the old files in the backup store. `flint obsidian update` and `repair` convert it too. `flint sync` blocks on it (exit 2) and names the conversion command. `flint open` opens it and prints the same command.
- When the vault is open or the fetch fails, the conversion waits (a pending conversion). `flint open` (after you close the vault) or the next `flint sync` completes it.

```bash
flint obsidian status                 # The payload (pinned, installed, available, pending), the vault, the plugin, and the next command [--check --tinderbox --json]
flint obsidian update                 # Pin the recommended release and install it; keep every personal setting [--to <release> --allow-downgrade --replace --tinderbox --only <names...> --dry-run --json]
flint obsidian repair                 # Restore the release files of the pinned release [--replace --reset-settings --file <path> --dry-run --json]
flint obsidian reload                 # Reload Obsidian for this Flint; list the terminal tabs that stop [--yes --json]
flint obsidian backups list|show|restore|prune           # The backup store of the Obsidian payload
flint obsidian profiles list|activate <id>|deactivate    # The appearance profiles of the installed release
flint obsidian applied [set|unset|apply]                 # The applied settings: each key, its layer, and its last result [--machine for the machine layer]
flint obsidian vaults repair          # Complete pending vault registry changes and remove stale entries; --yes writes [--yes --json]
flint open ["<name>"]                 # Open or focus a Flint in Obsidian [--no-launch]
flint obshn <orbh-profile> [prompt]   # Open a terminal tab in Obsidian and launch flint orbh i <orbh-profile> [--account <name> --at <scope> -p <flint>]
flint config set obsidian on|off      # The machine choice: Obsidian on or off on this machine
flint config set obsidian-trust auto|ask   # The trust setting: flint open trusts a checked Flint (auto), or Obsidian asks (ask)
```

Each `flint obsidian` command takes `-p <dir>` for another Flint. The commands `flint obsidian push`, `reset`, and `patches` are removed: each prints the new command. `profiles apply` is now `profiles activate <id>`.

## Send / Inbox

```bash
flint send <target> <title> <instructions> <files...>  # Send files to another Flint's inbox as a bundle
flint send "My Other Flint" "Auth Migration" "Move tasks to Tasks/" file1.md file2.md
```

Files land in the target Flint's `Inbox/(Bundle) Title/` directory. Use [[dev-sk-f-inbox_process]] to process incoming bundles.

## Other Useful Commands

```bash
flint open                            # Open or focus a Flint in Obsidian (see § Obsidian)
```
