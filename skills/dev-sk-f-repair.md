---
description: "Repair the failure of a Flint command from its failure record, with the person, and write one repair note in the Computer Flint"
---

> [!important] THIS FILE IS AN INSTRUCTION. WHEN REFERENCED IT IS MEANT TO BE TAKEN AS AN ACTION.

Run `flint shard start f` if you did not run it yet.

# Skill: Repair a Failure

A Flint command failed. `flint fix` started this repair session and gave you the failure record. Find the cause, repair it with the person, and write one repair note.

# Input

- The failure record: a JSON file in `~/.nuucognition/flint/failures/` (in `$NUU_HOME/flint/failures/` when `NUU_HOME` is set). The prompt of the session gives its path and its content. `flint fix --list` lists the records.
- The failed Flint: the field `flintPath` of the record. It is `null` when the command ran outside a Flint.
- The Flint of this session: the prompt names it. It is the Computer Flint. When the machine has no Computer Flint, it is the failed Flint; when the failure has no Flint either, it is the Flint of the working folder of `flint fix`. When a Flint cannot select an agent, `flint fix` passes the session to the next Flint of this order, and the prompt says why.

The failed Flint is usually outside the working folder of this session. Use absolute paths for each file and each command there. The harness can ask the person for permission for each edit outside the working folder. That is expected: tell the person why you edit the file.

The record has these fields: `command`, `args`, `exitCode`, `code`, `reason`, `next` (the `Next:` commands of the failure), `stderrTail`, `stdoutTail`, `flintVersion` (the version of the Flint CLI), `flintPath`, `cwd`, and `time`. Flint removed the secrets before it wrote the record.

# Rules

1. Ask the person before each change to the Git history, to the lock (`flint.json`), or to `flint.toml`. Say what you will change and why. Wait for a yes. These commands change them:
   - A commit, a reset, a rebase, a merge, and a push.
   - `flint git sync`, with or without a flag: it commits and pushes.
   - `flint shard install` and `flint shard update`: they move the lock.
   - `flint migrate run`: it writes `flint.json`.
2. Change nothing before you reproduce the failure. First use a command that writes nothing.
3. Prefer the `next` commands of the record. They are the repair that Flint gives. Use another command only when the record has none, or when the reproduction shows that it does not apply.
4. Run each command in the failed Flint: `cd '<flintPath>'`, or give `--path '<flintPath>'` to a command that has it.
5. Never delete a Flint, a source, or a Git branch to make a failure go away.
6. Tell the person the result of each step in one or two short sentences.
7. Never print a remote URL: it can hold a token. Name the remote (`origin`) only. To read the remotes, use `git -C '<flintPath>' remote`, not `git remote -v`.

# Actions

1. Read the failure record. Name the command that failed, its exit code, its `reason`, and its `next` commands. Tell the person what failed.
2. Read the state with no write:
   - `flint status` in the failed Flint (the versions and the migrations).
   - `flint obsidian status` in the failed Flint (the Obsidian payload).
   - `flint doctor -p '<flintPath>'`, or `flint doctor` when `flintPath` is `null` (the machine, the Computer Flint, the Flint registry, and the failed Flint: the lock, the sync plan, the Git state, the Obsidian payload). Its checks write nothing. When a row needs judgement (its next command is `flint fix`), it writes one new failure record: ignore that record, and do not start `flint fix` from this session.
3. Reproduce the failure with no write: `flint sync --dry-run` or `flint plan` in the failed Flint. For a failure of `flint git sync`, run `flint git status`. For a failure of `flint tinderbox sync`, run `flint tinderbox sync --dry-run` in the box folder.
4. Find the cause. Use the table below. When the cause is not in the table, read the tails of the record and the output of step 3.
5. Tell the person the cause and the repair that you propose: the commands, and each file that changes. Ask before each change that rule 1 names.
6. Do the repair. Prefer the `next` commands of the record.
7. Run the failed command again. When it fails again, go back to step 3. Stop after three tries, and tell the person what is still wrong.
8. Write one repair note in the Computer Flint (see "The repair note").

# Causes and Repairs

| Failure | Read first | Repair |
|---------|------------|--------|
| The lock does not match `flint.toml` (a declared shard is not installed) | `flint shard list` | `flint shard install`. It moves the lock: ask first. |
| `flint sync` stops on a content blocker (`flint.toml` cannot be parsed, a bad declaration) | `flint plan` | Correct the line that the blocker names. Ask before the edit of `flint.toml`. Then `flint sync`. |
| The Flint is behind the current Flint Version | `flint migrate run --dry-run` | `flint migrate run`. It writes `flint.json`: ask first. Then `flint sync`. |
| A shard migration is pending | `flint shard migrate show <alias>` | Follow the skill `sk-f-migrate`. |
| `flint git sync` halted on a conflict | `flint git status` | Show the person each conflict. Resolve with the person (`flint git resolve`), then `flint git sync --continue`. Or `flint git sync --abort`. Ask before each one. |
| `flint git sync` cannot reach origin, or the login failed | `git -C '<flintPath>' remote` (the names only) | Check the network and the Git login. Ask, then run `flint git sync` again. |
| The Obsidian payload (`.obsidian`) is missing or not current | `flint obsidian status` in the failed Flint | Run the next command that `flint obsidian status` gives. Ask before a command that replaces `.obsidian`. |
| `flint shard install` cannot reach a source (no network) | the `stderrTail` of the record | Check the network. Ask, then run `flint shard install` again. |
| `flint create` stopped after the folder was made | the `next` of the record | `flint create --resume '<path>'`. |
| `flint setup` failed | the `reason` of the record | Run the `next` command of the record. `flint whoami` shows the identity. |

# The Repair Note

Write one note in the Computer Flint with the Invironments shard. Run the command in the Computer Flint. When the machine has no Computer Flint, write the note in the failed Flint, and tell the person that `flint setup computer` makes the Computer Flint.

1. Write the body to a temporary file. The body has four sections:
   - `# What failed`: the command, its exit code, the failed Flint, and the time of the record.
   - `# The cause`: one or two sentences.
   - `# What changed`: each file and each Git change. Write "Nothing" when nothing changed.
   - `# The commands`: each command that ran, in order, in a code block.
2. Make the note:

   ```bash
   flint shard ie note "Repair - <command> - <YYYY-MM-DD HHMM>" --description "<one line: what failed and the cause>" --body-file <temporary file>
   ```

   The note goes to the New section. Its title names the failed command, for example `Repair - flint sync - 2026-09-27 1430`.
3. Tell the person the title of the note.

# Output

- The failed command runs with no failure, or the person knows what is still wrong and why.
- Each change to the Git history, the lock, or `flint.toml` had the yes of the person.
- One repair note in the Computer Flint (or in the failed Flint when the machine has no Computer Flint).
