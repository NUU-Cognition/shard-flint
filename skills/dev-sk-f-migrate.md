---
description: "Run a shard migration file against the workspace, then mark it finished"
---

> [!important] THIS FILE IS AN INSTRUCTION. WHEN REFERENCED IT IS MEANT TO BE TAKEN AS AN ACTION.

Run `flint shard start-dev f` if you haven't already.

# Skill: Run Shard Migration

Execute an agent-type shard migration — read the migration file, apply its transformations to the workspace, then mark it finished so the CLI can advance.

# Input

- Path to the migration file (e.g. `Shards/Flint/migrations/mig-f-0.1.3-to-0.1.4-s1.md`)
- The alias of the shard (e.g. `flint`) — the CLI takes a `<ref>`: the alias, the shorthand, the address, or the id. A folder name is not a ref.

# Actions

1. Read the migration file at the given path. Its body is a self-contained instruction set. When the file has a `rewrite` block (a shorthand rename), `flint shard migrate run <alias>` already rewrote the tags, the links, and the command texts in the Mesh, and it listed each file that it changed. Check those files; the file tells you what stays for you (the prose, a bare tag, a Markdown link, a code block).
2. Apply every transformation described. Migration files describe file renames, frontmatter edits, content rewrites, or structural moves — do them all, in order.
3. If the file has a `## Verification` section, run those checks. If anything fails, stop and report — do not finish.
4. Run `flint shard migrate finish <alias>` to mark the step complete. The CLI advances the version (if this was the last step of a transition) and surfaces the next pending step. The migration file stays in the build; the lock ledger records the step.
5. If `flint shard migrate show <alias>` reports more pending steps, the user can invoke this skill again with the next file path.

# Output

- Workspace reflects every change the migration file described.
- `flint shard migrate finish` has run successfully.
- The pending step is no longer in `flint shard migrate show`.

# Notes

- Migration files do not carry an `id:` field in frontmatter. The filename IS the canonical id — never invent or add one.
- `type: agent` in frontmatter means an agent (you) applies it. `type: manual` means the user does; you only run this skill for agent-type migrations.
- `type: script` migrations are auto-executed by the CLI and never reach this skill.
- Do not `finish` a step you could not fully apply. Report the gap instead so the user can decide.
