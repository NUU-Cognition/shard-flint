# Flint Releases

## Release 0.3.1

- `inst-f-flint_init` (the starter `Mesh/(System) Flint Init.md` of a new Flint): the title is `# New Flint`, and step 4 tells the agent to write the name of the Flint as the title (the install has no placeholder for the name). The file names the shards of each preset (`blank`, `default`, `factory`, `garden`, `lab`, `observatory`, `quarry`) and only commands that `flint --help` lists. It says that Invironments comes with Flint. It has no `#readonly` tag (NUU Flint Task 1048, UX7).
- `init-f`: the Identity section says `flint setup` (your Name and the machine names, once per machine), `flint config name` (your Name only), `flint create` (`init` is an alias), and the local Flint: an address that starts with `@/` names no org and works only on this machine; `flint org set <org>` shares the Flint. The presets are described as `flint create` installs them. The terms of Task 1048 UX10: your Name, the NUU config, the NUU home, the shard registry, local.
- `knw-f-cli`: a new section "Identity and New Flints" (`flint setup`, `flint whoami`, `flint config name`, `flint create`, `flint org show`, `flint org set`); `search` is the one alias of `browse`; the empty code block is gone.
- `knw-f-artifacts`, `knw-f-tinderbox`, `sk-f-stitch`: the terms of UX10 (the Name, the NUU config, "local" for no org).
- `init-f`, `knw-f-cli`, `sk-f-shard_browse`, `inst-f-flint_init`, README: `flint shard browse` and `flint shard install --core` are in the CLI again, on the package model (NUU Flint Task 1047). Browse lists the shard registry and the shard sources of the Flints of this machine (no GitHub scan). Every new Flint gets the core shards, whatever its preset declares.
- Requires a flint CLI with `flint shard browse` and `flint shard install --core` on the package model (the `nus-nuu1` sync of canon `78b5b1ff`).

## Release 0.3.0

- New skill `sk-f-shard_browse`: browse the shard catalog, suggest shards for the Flint's purpose, install the operator's picks.
- `init-f`: new "Choosing Shards" rules — core shards come with every Flint; browse before you install or create a shard.
- `knw-f-cli`: documents `flint shard browse` (public registry + GitHub + local dev shards) and `flint shard install --core`.
- `tmp-f-flint_init`: new "Shards" table (installed shards and why).
- `inst-f-flint_init`: the default Flint Init now tells the first agent to ask the purpose, browse, and install.
- Requires a flint CLI that has `flint shard browse` and `flint shard install --core`.

