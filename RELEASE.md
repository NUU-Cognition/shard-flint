# Flint Releases

## Unreleased

- `init-f`, `knw-f-cli`, `sk-f-shard_browse`, `inst-f-flint_init`, README: `flint shard browse` and `flint shard install --core` are in the CLI again, on the package model (NUU Flint Task 1047). Browse lists the registry and the shard sources of the Flints of this machine (no GitHub scan). Every new Flint gets the core shards, whatever its preset declares.
- Requires a flint CLI with `flint shard browse` and `flint shard install --core` on the package model (the `nus-nuu1` sync of canon `78b5b1ff`).

## Release 0.3.0

- New skill `sk-f-shard_browse`: browse the shard catalog, suggest shards for the Flint's purpose, install the operator's picks.
- `init-f`: new "Choosing Shards" rules — core shards come with every Flint; browse before you install or create a shard.
- `knw-f-cli`: documents `flint shard browse` (public registry + GitHub + local dev shards) and `flint shard install --core`.
- `tmp-f-flint_init`: new "Shards" table (installed shards and why).
- `inst-f-flint_init`: the default Flint Init now tells the first agent to ask the purpose, browse, and install.
- Requires a flint CLI that has `flint shard browse` and `flint shard install --core`.

