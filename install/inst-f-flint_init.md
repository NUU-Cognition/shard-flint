---
id: {{uuid}}
tags:
  - "#f/init"
---

# My Flint

This is a new Flint. The shards of its preset are installed (`flint shard list`). The purpose of this Flint is not set.

## First Session

Agent: do these steps in the first session, in this order.

1. Ask the operator one question: "What is this Flint for?" Do not guess.
2. Run `flint shard list` (what this Flint has) and search the registry site `https://shards.nuucognition.com/registry` (what exists). `flint shard browse` is not in this build of the CLI (planned).
3. Follow [[sk-f-shard_browse]]: suggest the shards that fit the purpose, and install the operator's picks with `flint shard install <spec>`.
4. Rewrite this file with [[tmp-f-flint_init-v0.1]]: the purpose of the Flint, the Shards table, and the navigation.

## Shards

Purpose not yet set — follow [[sk-f-shard_browse]] once the operator states the purpose.

## Navigation

- `Mesh/` — All workspace content
- `Media/` — All media documents
- `Shards/` — Installed capabilities (`flint shard list`)
