---
description: "Shard invocations — the record of one use of a shard, a workflow run with its steps and outputs, the stack of a session, the next hints, readiness and the ways to make an artifact ready, continuation, the next workflow, assumed states, modes, the Page block, Steel, and the refusals with their ways out"
orbh-sessions:
  - "[[d0c32612-4481-4a28-b4d5-75859b6e590e]]"
  - "[[ec13c5bf-9eab-4982-b35a-cffc13d48950]]"
---

# Knowledge: Shard Invocations

How to use shard invocations. Read this file before you run `flint shard invoke <ref> <workflow>`, when a workflow has an "Invocation" section, and when an invoke refuses because an artifact is not ready. To write an invocation file for a shard, load the Knap shard and read [[knw-knap-invocations]]. The lifecycle verbs of a session (`flint orbh close`, `flint orbh session return`) and the delivery of the Page are in the Orbh shard.

## What an invocation is

An invocation is the durable record of one use of a shard. Each record is one folder, `.flint/invocations/<id>/`, with one append-only file, `events.jsonl`. The id is `inv-` and 12 hex characters. `invocations show`, `set-input`, `set-output`, `close`, `unlock`, and `step --invocation` also take a prefix of 7 or more characters; `invoke --parent` needs the full id. Git ignores `.flint/`, so the records stay on this machine. Each state comes from the events of the record. Do not edit a record by hand.

There are three kinds:

| Kind | Made by | The record |
|---|---|---|
| `load` | `flint shard start <ref>`, `flint shard invoke <ref>` | It opens and closes at once, with the outcome `loaded`. It has no steps and no parent. |
| `workflow` | `flint shard invoke <ref> <workflow>` | It stays open until it closes. It holds the inputs, the steps, the outputs, the parent, the children, the behaviours that ran, and (format 0.3) the mode, the identity and the readiness of each artifact, the link to an earlier invocation, and the assumed states. |
| `assumption` | `flint shard assumed set`, `flint shard assumed withdraw` | It opens and closes at once. It holds one assumed state that a person set or withdrew. |

A workflow run needs an invocation file beside its Markdown workflow: `workflows/wkfl-<sh>-<name>.invocation.json`. One file serves both modes ([Modes](#modes)). The section `## Invocations` of the manifest of `flint shard start <ref>` lists each workflow that has one, with its `invoke` command.

An invocation file of format `shard-invocation/0.3` adds the sequence over time. One invocation leaves an artifact in a state, and that state makes the artifact ready for the next workflow, maybe in another session. The artifact carries the link: the **ready** condition of each input, the **next workflow** after a `done` close, the **state entry** at the open, the **continuation** of an earlier invocation on the same artifact, and the **assumed states**.

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
flint shard invocations --artifact '<artifact>'  # The invocations that took or made an artifact, oldest first, with the continues links
flint shard ready '<artifact>'                   # The workflows of the installed shards that take the artifact, with each verdict
flint shard assumed '<artifact>'                 # The assumed states of the artifact, with the source and why each holds or not
flint shard invocations reindex                  # Rebuild the index of the artifacts (a cache)
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
                 [--toward '<shard> <workflow>'] [--fresh]     # The target of a make-ready way; copy no earlier step
                 [--not-ready '<why>'] [--mode <mode>]         # Open on an artifact that is not ready (the person asks); the mode for a test
flint shard step                                               # The current step and the hint
flint shard step done <step> --evidence '<what proves it>'     # Settle the current step as done
flint shard step skip <step> --reason '<why>'                  # Settle the current step as skipped
flint shard invocations set-input [<id>] <name>=<value>        # Set or replace an input
flint shard invocations set-output [<id>] <name>=<value>       # Set an output (a return value)
flint shard invocations close [<id>] --outcome done            # Close it; or --outcome abandoned --reason '<why>'
```

### Open

`flint shard invoke <ref> <workflow>` opens a workflow invocation. The workflow is the short name (`do_task`) or the stem (`wkfl-proj-do_task`). `--dev` loads the source. The mode comes from the Orbh session ([Modes](#modes)).

For a 0.3 file, the open does these things in order, before the agent starts:

1. **Readiness.** It checks each artifact input that has a value. When one is not ready, it refuses, and it writes nothing ([Readiness](#readiness-when-an-artifact-is-not-ready)).
2. **Continuation.** It links to the newest invocation that took or made the same artifact, and it can copy the settled steps of that invocation ([Continuation](#continuation)).
3. **The state entry.** It settles each leading step that the state of the artifact proves done.
4. **The mode.** It skips each step that is not for the mode of the invocation.

- The output is the manifest of the shard, then `## Workflow invocation`: the id, the workflow, the file, the parent, the depth and the root, the mode, the link and the copy, the inputs with the verdict of each artifact input, the steps with the steps that the state and the mode settled, the outputs, the opt-in behaviours, and the hint lines.
- An input that the file does not declare refuses (`input-unknown`). A required input that is missing does not refuse: the output names `flint shard invocations set-input <id> <name>='<…>'`.
- A workflow with no invocation file refuses (`invocation-file-missing`). The refusal lists the workflows that have one.
- The behaviours of `invocation.open` run. For example, `task-open` of Projects sets a `todo` task to `in-progress`.
- `invoke` opens no Orbh procedure. Use `flint shard step`, not `flint orbh procedure step`.
- In an Orbh session, run `invoke` in the Flint of the session. An invoke that writes to another Flint refuses (`invocation-flint-mismatch`), because the behaviours of the session read only the Flint of the session.
- A 0.1 or 0.2 file has no readiness check, no continuation, no state entry, and no next workflow.

### Steps

- **In order.** Only the current step can settle: the first step that is not done or skipped. Each step before it must be done or skipped.
- **Evidence.** A step with `evidence required` needs `--evidence`. Write what proves the step: a commit, a file, a test result. A step with `evidence optional` takes it when you have it.
- **Skip.** `step skip` needs `--reason`. When a step does not apply, skip it with a reason. Do not mark it done with no work.
- **Repeat.** A `repeat` step (for example `converse` of a notepad) stays current until you mark it done. Do each turn of the loop inside it. Mark it done only when the loop ends.
- **Behaviours.** `step done` runs the behaviours of the place `step.done:<step>`. A `before` behaviour can refuse the step, and then nothing is written. An `after` behaviour that fails gives a warning; the step stays done. `step skip` runs no behaviour.
- **Fixed.** A settled step does not change (`step-settled`).

### Inputs and outputs

- `set-input` sets or replaces an input of an open invocation. The name must be an input of the file (`input-unknown`). For an artifact input of a 0.3 file, `set-input` checks the artifact as `invoke` does, before any write. `--not-ready '<why>'` sets it anyway (only when the person asks).
- `set-output` sets a return value. A second `set-output` of the same name replaces the value. An empty value counts as not set. The name must be an output of the file (`output-unknown`).
- A child output does not go into the parent output by itself. The parent record gets the outputs of the child in its `child.closed` event: `flint shard invocations show <parent id>` lists them under `Children:`. When the parent declares the same output, set it with `set-output`.

### Close

- `--outcome done` refuses while a step is not done or skipped (`steps-open`), and while a required output is not set (`outputs-missing`).
- `--outcome abandoned` closes with open steps. Give `--reason '<why>'`.
- A parent cannot close while a child is open (`children-open`). Close each child first. Or give `--abandon-children '<why>'`: each open child closes first, the deepest first, with the outcome `abandoned` and this reason.
- A closed invocation does not change: its steps, inputs, and outputs are fixed (`invocation-closed`).
- When a session closes (`flint orbh close`) or finishes a turn (`flint orbh session return --finish`), Flint closes each open frame of the session, from the top of the stack down, with the outcome `session-closed` or `session-returned`. A child in another session stays open, with a warning. `return --await` closes no frame.
- Only a `done` close writes the assumed states of the file and prints the next workflow. When the workflow is finished, close it with `--outcome done` yourself, before the session close or the return.

### The id

In an Orbh session, `step`, `set-input`, `set-output`, and `close` act on the top of the stack of `ORBH_SESSION_ID` when you give no id. To act on another frame, name it: `--invocation <id>` for `flint shard step`, and the id for `set-input`, `set-output`, and `close`.

Outside an Orbh session (no `ORBH_SESSION_ID`), there is no stack. Give `--invocation <id>` to `flint shard step`, and the id to `set-input`, `set-output`, and `close`. `invoke` with no `--parent` starts a new chain; give `--parent <id>` to open a child. The hints print these flags for you.

### The lock

Each write takes the lock of the record for a short time. When a stopped process leaves its lock, the write refuses with `lock-held`, and the next command is `flint shard invocations unlock <id>`. Run `unlock` only after that refusal, and only when no other process writes to that invocation. Then run the write again.

## Readiness: when an artifact is not ready

In a 0.3 file, an artifact input can declare **ready**: the state that the workflow needs before it starts. An artifact **qualifies** for the workflow when its state meets that condition. `invoke` checks each artifact input that has a value, before any write:

1. The value names one Mesh file (a title, a wikilink, or a path under `Mesh/`). Else `input-not-found`, `input-ambiguous` (two files), or `input-unreadable`.
2. The `(Type)` prefix of the file name equals the `type` of the input. Else `input-wrong-type`.
3. The state of the artifact meets `ready`. Else `input-not-ready`.

**The verdicts.** The output of `invoke` prints one verdict for each artifact input:

| Verdict | Meaning |
|---|---|
| `ready` | The frontmatter meets the condition. |
| `ready (assumed)` | The condition passes through an **assumed state**, and the line names its source: `(Task) 750 … is assumed scoped by proj scope_task inv-…, while status todo`. See [Assumed states](#assumed-states). |
| `not ready` | A part of the condition fails. `invoke` refuses. |

**The refusal.** `input-not-ready` writes nothing. It gives the state now, the state that the workflow needs, and the **ways** to make the artifact ready:

```
$ flint shard invoke proj review_task --input task='(Task) 746 Parallel Auth Refresh'
✘ (Task) 746 Parallel Auth Refresh is in-progress. proj review_task needs: status review.
  Next: flint shard invoke proj do_task --input task='(Task) 746 Parallel Auth Refresh' --toward 'proj review_task'
  Or:   flint shard invoke proj review_task --input task='(Task) 746 Parallel Auth Refresh' --not-ready '<why>'
  Tell the person what (Task) 746 Parallel Auth Refresh needs, and ask before you make it ready.
```

- A way is a workflow that moves the artifact forward (an `invoke` command with `--toward`), a script of the shard that you run by hand, or a line that says what to do (for example, "make a new task that continues it").
- The `--not-ready '<why>'` command opens the invocation anyway. The record keeps the reason.
- The last line depends on the mode of the session.

**The doctrine.** Follow these rules when `invoke` or `set-input` refuses an artifact:

1. **Do not edit the artifact to pass the check.** Do not change a status, a field, or an assumed state so that the condition is true. The condition says that the work of an earlier stage is missing. A changed field hides that.
2. **In an interactive session, explain and ask.** Tell the person three things: the state of the artifact now, the state that the workflow needs, and each way (with what it does). Ask which way to take. Then run the way that the person picks. When the way is a workflow, run its command as printed: `--toward` makes its close continue to the first workflow.
3. **In a headless session, do not choose.** No person can pick a way. Return the refusal as the blocker: a subagent puts the reason and the ways in its return to its dispatcher. A headless root can also use `flint orbh approval request "<title>" --body "<the reason and the ways>"`, and continue with the way of the verdict.
4. **`--not-ready '<why>'` only when the person asks.** Pass it only when the person asks for it in the current message, as for `--with task-confirm`. Write the words of the person as the reason.

The other refusals of an input say that the value is wrong, not the artifact: a title with a typo (`input-not-found`), a title of two files (`input-ambiguous`: give the path), or an artifact of another type (`input-wrong-type`). Fix the value and invoke again.

## Toward

`--toward '<shard> <workflow>'` names the workflow that this invocation makes the artifact ready for. The `invoke` way of a refusal adds it for you. The record keeps the target, and while the frame is open, the output of `invoke`, `flint shard step`, and the Page show a `toward:` line. When the frame closes `done`, the close checks the target first: when the artifact now qualifies, `next:` is the command of the target.

A workflow can do less when it goes toward a target. For example, `do_task` toward `proj review_task` does not ask for the confirm of the person: it sets the task to `review`, skips its step `close`, and closes `done`, so that a reviewer takes the task. Read the `# Invocation` section of the workflow when you see a `toward:` line.

## Continuation

At the open, `invoke` finds the newest invocation, of any session, open or closed, that took or made the same artifact. The link uses the Mesh `id` of the artifact, so a rename does not break it. The record keeps the link (`opened.continues`), and the output of `invoke` names it:

```
$ flint shard invoke proj do_task --input task='(Task) 745 Fix Stale Context % in Shell Chrome'
✓ opened inv-… · continues inv-6431ac116ad7 (proj create_and_do_task, session-closed, session 1651d8ef)
  settled by the state: work, report (task.status = review)
now:     step 3/4 review
read:    Stage 3: Task Review of Shards/Projects/workflows/wkfl-proj-do_task.md
```

**What `invoke` copies, and why.** When the earlier invocation is of the same workflow and it did not close `done` (for example, its session closed in the middle of the work), `invoke` copies its settled steps, in order from the first, and its outputs (`via: continued`). The earlier session did that work, and the record proves it. Thus a new session continues from the first open step, and it does not do the work again. The output says what it copied. Read the stages of the copied steps when you need their context: the work is in the artifact and in the outputs, not in your context.

- **`--fresh`** copies no settled step or output from the earlier invocation. The state entry and the mode skips still apply, so the new invocation can start after step 1. The link stays. Use it when the person wants a new pass, or when the earlier work is wrong.
- **`continue-open`.** When the earlier invocation is still open in a live session, `invoke` refuses: two sessions must not work on one artifact at the same time. Find that session (`flint shard invocations --artifact '<artifact>'`). When it still works on the artifact, stop, and tell the person. When it does not, close its invocation (`flint shard invocations close <id> --outcome abandoned --reason '<why>'`), or invoke with `--fresh`.
- An earlier invocation that closed `done`, or of another workflow, gives only the link: nothing is copied.

**The state entry.** After the copy, `invoke` settles each leading step whose `done-when` the state of the artifact meets (`via: state`, with the evidence, for example `task.status = review`). Thus `do_task` on a task in `review` starts at the step `review`, also when a person did the work by hand. Check that the state is true before you continue: read the artifact, and when the work is not there, tell the person.

**The thread of an artifact.** `flint shard invocations --artifact '<artifact>'` lists the invocations that took or made the artifact, oldest first, with the `continues` links: for example `scope_task` (session A), `do_task` (session B), `review_task` (session C). The list reads an index. The index is a cache: when a list or a link misses an invocation, run `flint shard invocations reindex`.

## The next workflow

When a frame closes `done`, the close checks the workflows that come next, and it prints one or two lines for each:

1. the `toward` target of the frame ([Toward](#toward)), then
2. each `then` workflow of the invocation file whose condition is true (for example, `do_task` has `then: review_task` only in the headless mode).

```
$ flint shard invocations close --outcome done
✓ proj scope_task · Invocation Readiness closed (done) · outputs: task = (Task) 750 Invocation Readiness
ready:   (Task) 750 Invocation Readiness is todo. It qualifies for proj do_task.
next:    flint shard invoke proj do_task --input task='(Task) 750 Invocation Readiness'
```

- A workflow that qualifies gets `ready:` and `next:` (the `invoke` command). A workflow that does not qualify gets `not ready:` with the reason and the ways.
- A frame with an open parent prints `back in:` first, and these lines after it.
- A close with another outcome (`abandoned`, `session-closed`, `session-returned`) prints no next workflow.
- **Do not run the next workflow by yourself.** In an interactive session, give the `next:` command to the person, and run it when the person asks. In a headless session, put it in the result: the dispatcher or the person starts it.

**What can come next.** `flint shard ready '<artifact>'` answers "what can I do with this artifact". It reads the invocation files of the installed shards, finds each workflow with an artifact input of the type of the artifact, and prints its verdict and the step where it starts (after the state entry):

```
$ flint shard ready '(Task) 745 Fix Stale Context % in Shell Chrome'
(Task) 745 … · Task · status review
  ✓ proj review_task     ready
  ✓ proj do_task         ready · it starts at step 3/4 review
```

`--json` gives `{ ok, artifact, workflows, assumed }`.

## Assumed states

An **assumed state** is a state that the record gives, not the frontmatter. A task in `todo` can be a stub, or a task that `scope_task` researched and specified. The status is the same, but only the second task is ready for work. When `scope_task` closes `done`, the record writes that the task is assumed `scoped`, while it is `todo`.

An assumed state holds while all of these are true:

- its guard (`while`) is true on the frontmatter now: when the task goes to `in-progress`, `scoped` stops by itself;
- no later invocation gave the same state again (the later one replaces it);
- nobody withdrew it.

```bash
flint shard assumed '<artifact>'                                    # Each state, its source, and why it holds or not
flint shard assumed set '<artifact>' <state> --reason '<why>'       # Set a state by hand (an assumption record)
flint shard assumed withdraw '<artifact>' <state> --reason '<why>'  # Withdraw a state (an assumption record)
```

A `ready` condition can name an assumed state. Then the verdict is `ready (assumed)`, and the line names the invocation that gave the state.

**When to set one by hand.** Set an assumed state only when the person says that the artifact is in that state, and no workflow gave it: for example, an old task that the person knows is scoped. Write the words of the person as the reason. Do not set a state to pass a readiness check on your own: that is an edit of the artifact to pass the check.

**When to withdraw one.** Withdraw a state when it is no longer true and its guard does not see it: for example, the person says that the scope of a `todo` task is wrong. Write why.

## Modes

An invocation runs in one of two **modes**: `interactive` or `headless`. The mode comes from the Orbh session: an interactive session gives `interactive`; a headless, subagent, or ping session gives `headless`. With no Orbh session, the mode is `interactive`. `--mode interactive|headless` sets the mode, for a test. The record keeps it, and the output of `invoke` prints it.

One workflow file serves both modes:

- **The callouts.** A stage that differs in a headless session has a `> [!headless]` callout inside that stage. In a headless session, do the normal text of the stage and follow its callout; when they conflict, the callout wins. An interactive session ignores the callouts. After a compaction, read the stage in `read:` again, with its callout.
- **The skipped steps.** A step only for a person (for example the confirm of a task) is skipped at the open in the headless mode (`via: mode`). `flint shard step` shows each step that the mode skipped. Do not settle it by hand.
- **The hints.** A step can have one `next:` text for each mode. The hint prints the text of the mode of the frame.
- **The conditions.** A behaviour, a next workflow, or a ready condition can depend on the mode. For example, `do_task` hands the task to `review_task` only in the headless mode.
- **A child** takes the mode of the session. Its command names the mode only when the mode of the frame differs from the mode of the session.

**Deprecated.** The headless files (`hwkfl-<sh>-<name>.md`, `hinit-<sh>.md`) and `flint shard hstart` go away (NUU Orbh Task 748). Until the removal wave: `invoke --headless` is an alias of `--mode headless`, with a notice; while a shard still has an `hwkfl-` file, the headless mode reads it first, with a notice; `hstart` prints a notice. After the removal wave, `--headless`, `hstart`, and `hstart-dev` are removed. A headless session runs `flint shard start <ref>` and follows the callouts.

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

Each write ends with hint lines: `invoke`, `step done`, `step skip`, `invocations set-input`, `set-output`, and `close`. `flint shard step`, `invocations show`, and `invocations stack` print them too. With `--json`, the commands are in `next` (an array). The close of a frame with no open parent prints no hint, unless the close is `done` and a next workflow applies: then it prints the lines of [The next workflow](#the-next-workflow).

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
| `toward:` | The workflow that this frame makes the artifact ready for (`--toward`). |
| `ready:` / `not ready:` | After a `done` close: the verdict of a next workflow. `next:` gives its command, or the ways follow. |

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
- A frame that goes toward another workflow gets a `toward:` line.
- `close:` and `return:` list the opt-in behaviours of the open frames, as `--with <id>`.
- A behaviour that failed gives `✘ <behaviour> failed`.
- With no open frame, the block is `INVOCATIONS  no open frame`.

**After a compaction.** The new context of an Orbh compaction reads the Page first. After a compaction of the harness (for example `/compact` of Claude Code), Orbh sends the Page one time. Its first line is `Context compacted: this is your Page. Read INVOCATIONS to continue where you are.` Then:

1. Read `you are in:`, the step of the top, and the `read:` line.
2. Read that stage of the Markdown workflow again.
3. Run `flint shard step` to see the steps and the hint.
4. Continue with the first `next:` line. Do not invoke the workflow again: the frame is still open.

**Steel.** The page **Invocations** (⌘9) lists the chains of the Flint and shows the story of one frame. The tab **Story** of a session (on the page Sessions) shows each event of the session in one column: the invocations as frames that open and close, their steps, the turns (open a turn to read it), the child sessions, and the compactions, with a detail panel. Both views update live. The story shows a `continues` edge from an invocation to the earlier invocation on the same artifact. The **thread** of an artifact shows its invocations across sessions, oldest first, with its assumed states and the workflows that it is ready for; the detail panel of an invocation lists "ready for". The Flint server gives the data: `GET /api/invocations`, `GET /api/invocations/:id`, `GET /api/invocations/:id/story`, `GET /api/sessions/:id/story`, `GET /api/artifacts/:id/thread` (the id is the Mesh `id` of the artifact), and the SSE channel `invocations`.

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
| `input-not-ready` | `invoke`, `set-input` | An artifact input does not meet the ready condition of the workflow. | Follow [the doctrine](#readiness-when-an-artifact-is-not-ready): in an interactive session, tell the person and run the way that the person picks (`Next:`); in a headless session, return the refusal as the blocker. `--not-ready '<why>'` only when the person asks. |
| `input-not-found` | `invoke`, `set-input` | The value of an artifact input names no Mesh file. | Fix the title, or give the path under `Mesh/`. For an artifact that a later stage creates, give no value now, and `set-input` it later. |
| `input-ambiguous` | `invoke`, `set-input` | The value names two Mesh files. | Give the path under `Mesh/`. |
| `input-unreadable` | `invoke`, `set-input` | The Mesh file cannot be read, or its frontmatter cannot be parsed. | Fix the frontmatter of the file, then invoke again. |
| `input-wrong-type` | `invoke`, `set-input` | The `(Type)` prefix of the file name is not the type of the input. | Give an artifact of the type that the workflow takes. `flint shard ready '<artifact>'` lists the workflows that take this one. |
| `continue-open` | `invoke` | The earlier invocation on the same artifact is still open in a live session. | Close that invocation (`flint shard invocations close <id> --outcome abandoned --reason '<why>'`) when its session does not work on it, or invoke with `--fresh`. |

## See also

- [[dev-knw-f-cli]] § Shard Invocations: the short command list.
- [[knw-knap-invocations]] (Knap shard): the invocation file, the steps, `calls`, outputs, behaviours, scripts, `ready`, `make-ready`, `then`, `done-when`, `assumes`, and `modes`, for an author.
- The Orbh shard (`Shards/Orbh/init-foh.md`): the lifecycle verbs, `--with`, the Page, and compaction.
- An example of the use side: the section "Invocation" of `wkfl-proj-do_task`, `wkfl-proj-scope_task`, `wkfl-proj-review_task`, `wkfl-ntpd-start`, and `wkfl-rpt-create`.
