---
description: "Browse the available shards, suggest the ones that fit this Flint's purpose, and install the operator's picks"
---

> [!important] THIS FILE IS AN INSTRUCTION. WHEN REFERENCED IT IS MEANT TO BE TAKEN AS AN ACTION.

Run `flint shard start-dev f` if you haven't already.

# Skill: Browse and Suggest Shards

Find the shards that exist before you build anything. Show the operator the catalog, suggest the shards that fit what this Flint is for, and install the ones the operator picks. Never create a new shard for a capability that an existing shard already provides.

# When to Use

- A new Flint: the operator has just told you what the Flint is for (the first session after `flint init`).
- The operator asks for a capability (tasks, notes, reports, meetings, a codebase map, ...).
- You are about to create a shard. Browse first. A duplicate shard splits the ecosystem.

# Input

- The purpose of this Flint, in the operator's words. If you do not have it, ask one question: "What is this Flint for?" Do not guess.
- Optional: a keyword to filter the catalog.

# Actions

1. Confirm the core shards are present:
   ```bash
   flint shard browse --local --available   # fast, no network; the Core Shards block is at the top of a full browse
   flint shard install --core               # only if a core shard is reported missing
   ```
2. Read the catalog:
   ```bash
   flint shard browse                       # public (registry + GitHub) and local dev shards, with install status
   flint shard browse <keyword>             # filter by name, shorthand, description, source, or Flint
   flint shard browse --available           # hide what this Flint already has
   flint shard browse --json                # machine-readable, same fields
   ```
   Read the `DESCRIPTION` column. The `SOURCE` column is the exact install argument. `STATUS` says what this Flint already has.
3. Match shards to the purpose. Use this table as the starting point, then add matches from the catalog descriptions:

   | Purpose of the Flint | Suggest |
   |---|---|
   | Any Flint | `NUU-Cognition/shard-invironments` (mesh sections and groups; a dependency of the Flint shard) |
   | Plan and track work | `NUU-Cognition/shard-projects`, `NUU-Cognition/shard-plan`, `NUU-Cognition/shard-increments` |
   | Think, brainstorm, take notes | `NUU-Cognition/shard-notepad`, `NUU-Cognition/shard-memory` |
   | Write reports, specs, guides | `NUU-Cognition/shard-reports`, `NUU-Cognition/shard-specifications`, `NUU-Cognition/shard-guides` |
   | Work on a codebase | `NUU-Cognition/shard-orbcode`, `NUU-Cognition/shard-code`, `NUU-Cognition/shard-codebase` |
   | Process meetings | `NUU-Cognition/shard-meetings` |
   | Teach or learn a topic | `NUU-Cognition/shard-learn` |
   | Make decisions with review | `NUU-Cognition/shard-proposals`, `NUU-Cognition/shard-mandate` |
   | Author shards | `NUU-Cognition/shard-knap` |
   | Drive a browser | `NUU-Cognition/shard-agent-browser`, `NUU-Cognition/shard-chrome-use` |

   Prefer a public shard over a local dev shard. Suggest a local dev shard (`flint://<Flint>/<shard>`) only when no public shard covers the need, and say that it is a working version from another Flint on this machine.
4. Present the suggestions to the operator as a short list: shard name, one line on why it fits, and the install source. Ask which ones to install. Do not install without the operator's pick.
5. Install the picks. Dependencies are installed with `--with-deps`:
   ```bash
   flint shard install <SOURCE> [<SOURCE>...] --with-deps
   ```
6. For every installed shard, run `flint shard start <name>` and read its init file before you use it.
7. Record the result in `Mesh/(System) Flint Init.md`: the purpose of the Flint and the shards installed for it. Follow [[dev-tmp-f-flint_init-v0.1]] and [[dev-sk-f-init_update]].

# Output

- The shards the operator picked are installed, with their dependencies.
- `Mesh/(System) Flint Init.md` names the purpose of the Flint and the installed shards.
- No new shard was created for a capability that an existing shard provides.
