# Flint Releases

## Release 0.3.2

- `shard.yaml`: the dependency on Invironments has its Git source again (`dependencies: - source: NUU-Cognition/shard-invironments`), as in 0.3.0 and in Orbh. 0.3.1 declared the package spec `"@nuu-cognition/invironments": ""`. A fresh machine does not have that package, and the shard registry does not serve it. Thus the install of 0.3.1 failed on every fresh machine (`dependency @nuu-cognition/shard/invironments of Flint cannot be resolved`), and Orbh then installed Flint 0.3.0 from the default branch (NUU Flint Task 1048, ruling 36). The Git source resolves with no shard registry.
- `init-f`, `knw-f-cli`, `knw-f-tinderbox`: the behaviour of `flint git`, `flint tinderbox`, `flint sync`, and `flint shard` of NUU Flint Task 1049. `flint sync` writes no shard version and no pin; its one lock write removes a stale lock line.
- The pin of the flint CLI moves to `@nuu-cognition/flint@0.3.2` (Task 1048, ruling 36). A CLI with the pin 0.3.1 installs 0.3.1 and gets the defect above.

## Release 0.3.1

- `inst-f-flint_init` (the starter `Mesh/(System) Flint Init.md` of a new Flint): the title is `# {{name}}`, the name of the Flint. The file names the shards of each preset (`blank`, `default`, `factory`, `garden`, `lab`, `observatory`, `quarry`) and only commands that `flint --help` lists. It says that Invironments comes with Flint. The source has no `#readonly` tag, and a CLI of NUU Flint Task 1048 adds none to a payload of `mode: once` (UX7, ruling 32).
- `init-f`: the Identity section says `flint setup` (your Name and the machine names, once per machine), `flint config name` (your Name only), `flint config org set` (the org of this machine), `flint create` (`init` is an alias), and the local Flint: an address that starts with `@/` names no org and works only on this machine; `flint org set <org>` shares the Flint. The presets are described as `flint create` installs them. The terms of Task 1048 UX10: your Name, the NUU config, the NUU home, the shard registry, local.
- `knw-f-cli`: a new section "Identity and New Flints" (`flint setup`, `flint whoami`, `flint config name`, `flint create`, `flint org show`, `flint org set`); `search` is the one alias of `browse`; the empty code block is gone.
- `knw-f-artifacts`, `knw-f-tinderbox`, `sk-f-stitch`: the terms of UX10 (the Name, the NUU config, "local" for no org).
- `init-f`, `knw-f-cli`, `sk-f-shard_browse`, `inst-f-flint_init`, README: `flint shard browse` and `flint shard install --core` are in the CLI again, on the package model (NUU Flint Task 1047). Browse lists the shard registry and the shard sources of the Flints of this machine (no GitHub scan). Every new Flint gets the core shards, whatever its preset declares.
- Requires a flint CLI with `flint shard browse` and `flint shard install --core` on the package model (the `nus-nuu1` sync of canon `78b5b1ff`), and a CLI that fills `{{name}}` in a payload (NUU Flint Task 1048, ruling 32). An older CLI prints the title `# {{name}}`.

## Release 0.3.0

- New skill `sk-f-shard_browse`: browse the shard catalog, suggest shards for the Flint's purpose, install the operator's picks.
- `init-f`: new "Choosing Shards" rules — core shards come with every Flint; browse before you install or create a shard.
- `knw-f-cli`: documents `flint shard browse` (public registry + GitHub + local dev shards) and `flint shard install --core`.
- `tmp-f-flint_init`: new "Shards" table (installed shards and why).
- `inst-f-flint_init`: the default Flint Init now tells the first agent to ask the purpose, browse, and install.
- Requires a flint CLI that has `flint shard browse` and `flint shard install --core`.

