---
id: {{uuid}}
tags:
  - "#f/init"
---

# New Flint

This is a new Flint. Its purpose is not set yet.

## Shards

Every new Flint has the core shards, Flint and Orbh, and Invironments (Flint needs it). The preset of `flint create` adds the other shards. Run `flint shard list` to see the shards of this Flint.

| Preset | The shards that the preset adds to the core shards |
|---|---|
| `blank` (the default) | None |
| `default` | Projects, Notepad, Plan, Increments, Reports, Agents, Claude Code |
| `factory` | Projects, Increments, Reports |
| `garden` | Notepad |
| `lab` | Notepad, Plan |
| `observatory` | Reports |
| `quarry` | Notepad |

## First Session

Agent: do these steps in the first session, in this order.

1. Ask the operator one question: "What is this Flint for?" Do not guess.
2. Run `flint shard list` (the shards of this Flint) and `flint shard browse` (the shards that this Flint can install: the shard registry and the shard sources of this machine).
3. Follow [[sk-f-shard_browse]]: suggest the shards that fit the purpose. Install the operator's picks with `flint shard install <spec>`.
4. Rewrite this file with [[tmp-f-flint_init-v0.1]]. The title is the name of this Flint (`name` in `flint.toml`). Write the purpose, the Shards table, and the navigation. Remove the tag `#readonly` if the install added it: this file belongs to the Flint.

## Navigation

- `Mesh/` — the notes of this Flint
- `Media/` — the files that are not Markdown (images, PDFs)
- `Shards/` — the installed shards (`flint shard list`)
