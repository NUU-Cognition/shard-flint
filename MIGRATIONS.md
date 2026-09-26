# Migrations

Migration notes for Flint. Latest version at top, separated by `---`.

# 0.3.6

- No migration step. The install replaces `Shards/(Shards) Obsidian Templates/otmp-f-default.md` when the lock records it and nobody changed it. The new copy has no `#readonly` tag.
- A copy that the lock does not record, or a copy that somebody changed, stays when it differs from the new note template. The install prints `Kept note template … Next: flint shard reinstall <alias> --replace-note-templates`. Run that command: it saves a backup of the old copy and installs the new note template.
- The Flint migration of 0.7.0 does not change note templates (NUU Flint Task 1055, decision S33 of `(Spec) Flint Obsidian`). The rollout of 0.7.0 does it in each Flint: run `flint shard update`, then run `flint shard reinstall <alias> --replace-note-templates`. No note changes.
