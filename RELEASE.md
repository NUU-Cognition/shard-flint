# Flint Releases

## Release 0.3.0

- New skill `sk-f-shard_browse`: browse the shard catalog, suggest shards for the Flint's purpose, install the operator's picks.
- `init-f`: new "Choosing Shards" rules — core shards come with every Flint; browse before you install or create a shard.
- `knw-f-cli`: documents `flint shard browse` (public registry + GitHub + local dev shards) and `flint shard install --core`.
- `tmp-f-flint_init`: new "Shards" table (installed shards and why).
- `inst-f-flint_init`: the default Flint Init now tells the first agent to ask the purpose, browse, and install.
- Requires a flint CLI that has `flint shard browse` and `flint shard install --core`.

