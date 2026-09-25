---
description: "Browse the available shards, suggest the ones that fit this Flint's purpose, and install the operator's picks"
---

> [!important] THIS FILE IS AN INSTRUCTION. WHEN REFERENCED IT IS MEANT TO BE TAKEN AS AN ACTION.

Run `flint shard start-dev f` if you haven't already.

# Skill: Browse and Suggest Shards

Find the shards that exist before you build anything. Show the operator the catalog, suggest the shards that fit what this Flint is for, and install the ones the operator picks. Never create a new shard for a capability that an existing shard already provides.

# When to Use

- A new Flint: the operator has just told you what the Flint is for (the first session after `flint create`).
- The operator asks for a capability (tasks, notes, reports, meetings, a codebase map, ...).
- You are about to create a shard. Browse first. A duplicate shard splits the ecosystem.

# Input

- The purpose of this Flint, in the operator's words. If you do not have it, ask one question: "What is this Flint for?" Do not guess.
- Optional: a keyword to filter the catalog.

# Actions

1. Confirm the core shards are present. `flint shard list` must have a row for `flint` and for `orbh`. Install a missing one:
   ```bash
   flint shard list                         # what this Flint has
   flint shard install --core               # installs each missing core shard; leaves the present ones alone
   ```
2. Read the catalog. It lists the shards of the registry and the shard sources of the Flints of this machine, each with its state in this Flint, its spec, and its next command:
   ```bash
   flint shard browse                       # the whole catalog
   flint shard browse <word> --available    # the shards that match a word and that this Flint does not have
   ```
   If `browse` prints a registry warning, it lists only the local sources. Then also search the registry site `https://shards.nuucognition.com/registry` by name or description. `flint resolve @nuu-cognition/shard/<slug>` checks one package: the rung (this Flint, this machine, or the registry), the address, and the tag.
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

   Prefer a public shard over a local source. Suggest a local source (`@org/<slug>#<flint slug>`) only when no public shard covers the need, and say that it is a working version from another Flint on this machine.
4. Present the suggestions to the operator as a short list: shard name, one line on why it fits, and the install source. Ask which ones to install. Do not install without the operator's pick.
5. Install the picks. The missing dependencies are installed too:
   ```bash
   flint shard install <spec> [<spec>...]
   ```
6. For every installed shard, run `flint shard start <alias>` and read its init file before you use it.
7. Record the result in `Mesh/(System) Flint Init.md`: the purpose of the Flint and the shards installed for it. Follow [[dev-tmp-f-flint_init-v0.1]] and [[dev-sk-f-init_update]].

# Output

- The shards the operator picked are installed, with their dependencies.
- `Mesh/(System) Flint Init.md` names the purpose of the Flint and the installed shards.
- No new shard was created for a capability that an existing shard provides.
