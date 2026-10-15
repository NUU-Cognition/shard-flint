---
description: "Shard invocations — the record of one use of a shard, a workflow run with its steps and outputs, the stack of a session, the next hints, the Page block, Steel, and the refusals with their ways out"
orbh-sessions:
  - "[[d0c32612-4481-4a28-b4d5-75859b6e590e]]"
---

# Knowledge: Shard Invocations

How to use shard invocations. Read this file before you run `flint shard invoke <ref> <workflow>`, and when a workflow has an "Invocation" section. To write an invocation file for a shard, load the Knap shard and read [[knw-knap-invocations]]. The lifecycle verbs of a session (`flint orbh close`, `flint orbh session return`) and the delivery of the Page are in the Orbh shard.

## What an invocation is

An invocation is the durable record of one use of a shard. Each record is one folder, `.flint/invocations/<id>/`, with one append-only file, `events.jsonl`. The id is `inv-` and 12 hex characters. `invocations show`, `set-input`, `set-output`, `close`, `unlock`, and `step --invocation` also take a prefix of 7 or more characters; `invoke --parent` needs the full id. Git ignores `.flint/`, so the records stay on this machine. Each state comes from the events of the record. Do not edit a record by hand.

There are two kinds:

| Kind | Made by | The record |
|---|---|---|
| `load` | `flint shard start <ref>`, `flint shard invoke <ref>` | It opens and closes at once, with the outcome `loaded`. It has no steps and no parent. |
| `workflow` | `flint shard invoke <ref> <workflow>` | It stays open until it closes. It holds the inputs, the steps, the outputs, the parent, the children, and the behaviours that ran. |

A workflow run needs an invocation file beside its Markdown workflow: `workflows/wkfl-<sh>-<name>.invocation.json`, or `hwkfl-<sh>-<name>.invocation.json` for a headless workflow. The section `## Invocations` of the manifest of `flint shard start <ref>` lists each workflow that has one, with its `invoke` command.

A workflow invocation is a function call of a shard. The inputs are the arguments. The steps are the body. The outputs are the return value. The parent is the caller. The open workflow invocations of one Orbh session are a call stack, and each one is a **frame**. The **top of the stack** is the frame that the session opened last.

The invocation file does not replace the Markdown workflow. Read the workflow and follow its stages. The record tells you where you are in it.

## See the invocations

```bash
flint shard invocations                          # The open invocations of this Flint (--open is the default)
flint shard invocations --all                    # Open and closed, the loads too
flint shard invocations --mine                   # Only the invocations of this Orbh session (ORBH_SESSION_ID)
flint shard invocations --session <id>           # Only the invocations of one Orbh session
flint shard invocations show <id>                # The summary, the steps, the outputs, the children, the events, and the hint
flint shard invocations stack [--session <id>]   # The frames of one session, the root first; "← top" marks the top
flint shard step [--invocation <id>]             # Where am I: the steps of one frame (default: the top of the stack) and the hint
```

Each command takes `--json` and `-p <dir>`.

```
$ flint shard invocations stack
Stack of the session 7fc4b42c-…: 2 frames, the root first.
ntpd start · Stack Test · step 3/4 converse (repeat)  inv-1ca76ca58901 · depth 0
└─ proj do_task · (Task) 001 Dry Work · step 1/4 work  inv-a9f1facfa8ea · depth 1  ← top
```

A frame has a label: `<shorthand> <workflow> · <the first input>`, for example `proj do_task · (Task) 744 …`.

## Run a workflow

```bash
flint shard invoke <ref> <workflow> --input <name>=<value>     # Open the invocation (repeat --input)
flint shard step                                               # The current step and the hint
flint shard step done <step> --evidence '<what proves it>'     # Settle the current step as done
flint shard step skip <step> --reason '<why>'                  # Settle the current step as skipped
flint shard invocations set-input [<id>] <name>=<value>        # Set or replace an input
flint shard invocations set-output [<id>] <name>=<value>       # Set an output (a return value)
flint shard invocations close [<id>] --outcome done            # Close it; or --outcome abandoned --reason '<why>'
```

### Open

`flint shard invoke <ref> <workflow>` opens a workflow invocation. The workflow is the short name (`do_task`) or the stem (`wkfl-proj-do_task`). `--headless` reads the headless file (`hwkfl-`) first. `--dev` loads the source.

- The output is the manifest of the shard, then `## Workflow invocation`: the id, the workflow, the file, the parent, the depth and the root, the inputs, the steps, the outputs, the opt-in behaviours, and the hint lines.
- An input that the file does not declare refuses (`input-unknown`). A required input that is missing does not refuse: the output names `flint shard invocations set-input <id> <name>='<…>'`.
- A workflow with no invocation file refuses (`invocation-file-missing`). The refusal lists the workflows that have one.
- The behaviours of `invocation.open` run. For example, `task-open` of Projects sets a `todo` task to `in-progress`.
- `invoke` opens no Orbh procedure. Use `flint shard step`, not `flint orbh procedure step`.
- In an Orbh session, run `invoke` in the Flint of the session. An invoke that writes to another Flint refuses (`invocation-flint-mismatch`), because the behaviours of the session read only the Flint of the session.

### Steps

- **In order.** Only the current step can settle: the first step that is not done or skipped. Each step before it must be done or skipped.
- **Evidence.** A step with `evidence required` needs `--evidence`. Write what proves the step: a commit, a file, a test result. A step with `evidence optional` takes it when you have it.
- **Skip.** `step skip` needs `--reason`. When a step does not apply, skip it with a reason. Do not mark it done with no work.
- **Repeat.** A `repeat` step (for example `converse` of a notepad) stays current until you mark it done. Do each turn of the loop inside it. Mark it done only when the loop ends.
- **Behaviours.** `step done` runs the behaviours of the place `step.done:<step>`. A `before` behaviour can refuse the step, and then nothing is written. An `after` behaviour that fails gives a warning; the step stays done. `step skip` runs no behaviour.
- **Fixed.** A settled step does not change (`step-settled`).

### Inputs and outputs

- `set-input` sets or replaces an input of an open invocation. The name must be an input of the file (`input-unknown`).
- `set-output` sets a return value. A second `set-output` of the same name replaces the value. An empty value counts as not set. The name must be an output of the file (`output-unknown`).
- A child output does not go into the parent output by itself. The parent record gets the outputs of the child in its `child.closed` event: `flint shard invocations show <parent id>` lists them under `Children:`. When the parent declares the same output, set it with `set-output`.

### Close

- `--outcome done` refuses while a step is not done or skipped (`steps-open`), and while a required output is not set (`outputs-missing`).
- `--outcome abandoned` closes with open steps. Give `--reason '<why>'`.
- A parent cannot close while a child is open (`children-open`). Close each child first. Or give `--abandon-children '<why>'`: each open child closes first, the deepest first, with the outcome `abandoned` and this reason.
- A closed invocation does not change: its steps, inputs, and outputs are fixed (`invocation-closed`).
- When a session closes (`flint orbh close`) or finishes a turn (`flint orbh session return --finish`), Flint closes each open frame of the session, from the top of the stack down, with the outcome `session-closed` or `session-returned`. A child in another session stays open, with a warning. `return --await` closes no frame.

### The id

In an Orbh session, `step`, `set-input`, `set-output`, and `close` act on the top of the stack of `ORBH_SESSION_ID` when you give no id. To act on another frame, name it: `--invocation <id>` for `flint shard step`, and the id for `set-input`, `set-output`, and `close`.

Outside an Orbh session (no `ORBH_SESSION_ID`), there is no stack. Give `--invocation <id>` to `flint shard step`, and the id to `set-input`, `set-output`, and `close`. `invoke` with no `--parent` starts a new chain; give `--parent <id>` to open a child. The hints print these flags for you.

### The lock

Each write takes the lock of the record for a short time. When a stopped process leaves its lock, the write refuses with `lock-held`, and the next command is `flint shard invocations unlock <id>`. Run `unlock` only after that refusal, and only when no other process writes to that invocation. Then run the write again.

## The stack

Each workflow invocation has a parent, a root (the first frame of its chain), and a depth (0 for a root). `invoke` chooses the parent in this order:

1. `--root`: no parent. The invocation starts a new chain.
2. `--parent <id>`: that invocation, in any session. It must be an open workflow invocation (`parent-not-found`, `parent-not-workflow`, `parent-closed`).
3. The top of the stack of the session. Thus an invoke inside an open notepad opens a child of the notepad, with no flag.
4. The parent of a child session. When a session with an open frame launches a child session (a dispatch, a fleet member), Flint gives the child `FLINT_PARENT_INVOCATION` (the top of the stack of the launcher) and writes `.flint/invocation-launches/<session id>.json`. The first invoke of the child takes that frame as its parent, also after a resume or a compaction.
5. Else, no parent.

- `--root` and `--parent` together refuse (`parent-conflict`). A load has no parent: `--parent` or `--root` with no workflow refuses (`parent-without-workflow`).
- The deepest frame has depth 8, so a chain has at most 9 frames (`depth-cap`). A loop of invocations can cause this.
- The parent record gets a copy of each child: `child.opened` (with the step of the parent at that time) and `child.closed` (with the outcome and the outputs). The child record is the truth.
- When a child closes, its parent is the current frame again. The hint of the close starts with `back in:`.

## Follow the hints

Each write ends with hint lines: `invoke`, `step done`, `step skip`, `invocations set-input`, `set-output`, and `close`. `flint shard step`, `invocations show`, and `invocations stack` print them too. With `--json`, the commands are in `next` (an array). The close of a frame with no open parent prints no hint: the chain is done.

```
$ flint shard step done work --evidence 'commit 3f2a1c9'
✓ proj do_task · (Task) 001 Dry Work · step 1/4 work done
now:     step 2/4 report
read:    Stage 2: Report of Shards/Projects/workflows/wkfl-proj-do_task.md
next:    flint shard invoke rpt create --input source='(Task) 001 Dry Work'
         Make a report when '(Task) 001 Dry Work' asks for one, … Else skip this step: …
         flint shard step done report
part of: ntpd start · Dry Chain · step converse
```

| Line | What it says |
|---|---|
| `now:` | The current step of the frame (`step <n>/<total> <id>`), or `all steps settled`. |
| `read:` | The stage of the Markdown workflow for the current step: `<stage> of <path>`. |
| `next:` | The next actions, one on each line. The first line is the primary action. |
| `part of:` | The parent frame and its current step. A root has no `part of:` line. |
| `back in:` | After a child closes: the open parent frame and its step. Its `read:`, `next:`, and `part of:` lines follow. |

The order of the `next:` lines: each child invocation that the step calls and that is not open yet (`flint shard invoke …`), then the instruction of the step, then the command that settles the step (`flint shard step done …`). When each step is settled: a `set-output` command for each required output that is not set, then `flint shard invocations close <id> --outcome done`.

The rules:

1. **Do the first `next:` line first.** It is the primary action. A later line is the action after it, or the other way.
2. **Run a command as printed.** Each value is in POSIX single quotes, so the shell expands nothing in it. Replace only a placeholder: `'<…>'` (a value, for example the evidence) and `'<why>'` (a reason). Keep the single quotes.
3. **Keep the frame flags.** A frame that is not the top of its stack, or a frame with no session, names itself: `--invocation <id>` on `flint shard step`, and `--parent <id>` on `flint shard invoke`. With no flag, the verb acts on the top of the stack.
4. **Read the stage.** Read the stage that `read:` names before you work on the step. After a compaction, read it again: the context is gone, but the record stays.
5. **A token with no value.** A `{{…}}` token in a `next:` line has no value yet, for example `{{output.notepad}}` before you set the output. Set the value; the next hint fills it.
6. **A step that does not apply.** A `calls` command is what the workflow expects at that step. When the work does not need the child (for example, the task needs no report), skip the step with a reason.

## The Page and Steel

**The Page.** The block `INVOCATIONS` of `flint orbh page` shows the stack of the session:

```
INVOCATIONS  you are in: proj do_task · (Task) 001 Dry Work
  ntpd start · Stack Test · step converse (repeat)          inv-1ca76ca58901 · 3h
  └─ proj do_task · (Task) 001 Dry Work · step 1/4 work ●   inv-a9f1facfa8ea · 40m
       read:  Stage 1: Task Work of Shards/Projects/workflows/wkfl-proj-do_task.md
       next:  Implement '(Task) 001 Dry Work': tick each checkbox, …
              flint shard step done work --evidence '<…>'
       close: --with task-confirm · --with task-defer
       return: --with task-confirm · --with task-defer
```

- `●` marks the top of the stack. The top gets the `read:` and `next:` lines of its hint.
- A frame whose parent is in another session gets a `part of:` line.
- `close:` and `return:` list the opt-in behaviours of the open frames, as `--with <id>`.
- A behaviour that failed gives `✘ <behaviour> failed`.
- With no open frame, the block is `INVOCATIONS  no open frame`.

**After a compaction.** The new context of an Orbh compaction reads the Page first. After a compaction of the harness (for example `/compact` of Claude Code), Orbh sends the Page one time. Its first line is `Context compacted: this is your Page. Read INVOCATIONS to continue where you are.` Then:

1. Read `you are in:`, the step of the top, and the `read:` line.
2. Read that stage of the Markdown workflow again.
3. Run `flint shard step` to see the steps and the hint.
4. Continue with the first `next:` line. Do not invoke the workflow again: the frame is still open.

**Steel.** The page **Invocations** (⌘9) lists the chains of the Flint and shows the story of one frame. The tab **Story** of a session (on the page Sessions) shows each event of the session in one column: the invocations as frames that open and close, their steps, the turns (open a turn to read it), the child sessions, and the compactions, with a detail panel. Both views update live. The Flint server gives the data: `GET /api/invocations`, `GET /api/invocations/:id`, `GET /api/invocations/:id/story`, `GET /api/sessions/:id/story`, and the SSE channel `invocations`.

## Behaviours and `--with`

An invocation file can declare **behaviours**: reactions that run at a place (`invocation.open`, `step.done:<step>`, `session.close`, `session.return`). They do part of the work of the workflow for you, for example a status change of a task. When the invocation is open, do not do that work by hand.

- An **opt-in** behaviour runs only when the verb names it: `flint orbh close --with <id>[=<value>]`, or `flint orbh session return --finish|--await --with <id>[=<value>]`. The output of `invoke` lists them under `Opt-in behaviours:`, and the Page lists them on the `close:` and `return:` lines.
- An unknown `--with` name stops the verb (`unknown-with`). A name whose condition is false prints `<id>: … Nothing done.`, and the verb continues.
- A behaviour marked `by a person` (for example `task-confirm` of Projects): pass it only when the person asks for it in the current message.
- `--no-behaviours` skips the behaviours of the verb. The record keeps the skip.
- A file with `enforced: true`: `flint orbh session return --finish` refuses (`enforced-steps-open`) while a step of that invocation is not done or skipped. Settle the steps, or pass `--no-behaviours`.

## Refusals

A refusal writes nothing. It prints `✘ <reason>`, then the commands that get you out: `Next:` first, then `Or:`. With `--json`, it is `{ ok: false, code, reason, next }`. Read the reason, then run the `Next:` command or an `Or:` command.

```
$ flint shard step done gather --invocation inv-fb7aa19ea2b0
✘ Steps settle in order. The step 2/4 gather of ntpd start · Dry Chain waits for 1. create. The current step is 1/4 create.
  Next: flint shard step done create --evidence '<…>' --invocation inv-fb7aa19ea2b0
  Or:   flint shard step skip create --reason '<why>' --invocation inv-fb7aa19ea2b0
```

| Code | Verb | Cause | The way out |
|---|---|---|---|
| `step-out-of-order` | `step done`, `step skip` | The step is not the current step: a step before it is not done or skipped. | Settle the current step first: `Next: flint shard step done <current> …`, or `Or: flint shard step skip <current> --reason '<why>'`. |
| `evidence-required` | `step done` | The step needs evidence, and `--evidence` is missing or empty. | `Next: flint shard step done <step> --evidence '<…>'` with what proves the step, or `Or:` skip it with a reason. |
| `reason-required` | `step skip` | `--reason` is missing or empty. | `Next: flint shard step skip <step> --reason '<why>'`. |
| `step-settled` | `step done`, `step skip` | The step is already done or skipped. | Settle the current step: the `Next:` commands. |
| `steps-open` | `invocations close --outcome done` | A step is not done or skipped. | `Next:` settles the first open step (`flint shard step done <step> --invocation <id> …`), `Or:` skips it, or `Or:` closes with `--outcome abandoned --reason '<why>'`. |
| `outputs-missing` | `invocations close --outcome done` | A required output is not set, or it is empty. | `Next: flint shard invocations set-output <id> <name>='<…>'`, then close again. Or close with `--outcome abandoned --reason '<why>'`. |
| `children-open` | `invocations close` | The invocation has an open child. | `Next:` gives one command for each open child (its next step, its missing output, or its close). Or close the parent with `--abandon-children '<why>'`. |
| `parent-closed` | `invoke` | The parent is closed, or it closed during the invoke. A closed invocation cannot get a child. | `Next: flint shard invocations` lists the open invocations. Invoke with `--parent <open id>`, or with `--root` for a new chain. |
| `frame-required` | `flint shard step` | No `--invocation`, and no `ORBH_SESSION_ID` or no open frame in the session. | `Next: flint shard invocations stack`, or give `--invocation <id>`. |
| `invocation-id-required` | `set-input`, `set-output`, `close` | No id, and no `ORBH_SESSION_ID` or no open frame in the session. | Give the id. `flint shard invocations` lists them. |
| `invocation-closed` | each write | The invocation is closed. | Nothing to change. `flint shard invocations show <id>` shows the record. |
| `enforced-steps-open` | `flint orbh session return --finish` | An enforced invocation of the session has a step that is not done or skipped. | Settle the step (`flint shard step done|skip … --invocation <id>`), or return with `--no-behaviours`. |
| `unknown-with` | `flint orbh close`, `flint orbh session return` | No open invocation of the session declares that opt-in name for that verb. | Use a name from the `close:` or `return:` line of the Page. |
| `lock-held` | each write | Another process holds the lock of the record, or a stopped process left it. | `Next: flint shard invocations unlock <id>`, then the write again. |
| `depth-cap` | `invoke` | The parent has depth 8. | Close frames of the chain, or invoke with `--root`. |

## See also

- [[dev-knw-f-cli]] § Shard Invocations: the short command list.
- [[knw-knap-invocations]] (Knap shard): the invocation file, the steps, `calls`, outputs, behaviours, and scripts, for an author.
- The Orbh shard (`Shards/Orbh/init-foh.md`): the lifecycle verbs, `--with`, the Page, and compaction.
- An example of the use side: the section "Invocation" of `wkfl-proj-do_task`, `wkfl-ntpd-start`, and `wkfl-rpt-create`.
