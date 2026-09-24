---
description: "Type install convention — source naming, destination naming, subtype handling, shard-context qualifier, folders decoupling"
---

# Knowledge: Type Install Convention

Complete reference for how shards declare, author, and install artifact type definitions. Follows the [[MKD Title ID Protocol]] for destination filenames.

## Overview

Every shard that manages an artifact type (Task, Notepad, Increment, etc.) ships a **type definition file** — a markdown document that describes the type's properties, lifecycle, templates, and naming conventions. These files live in the `install/` folder of the source and of the shard, and the install (or the build) copies them into `Mesh/Metadata/Types/`.

The `types:` field in `shard.yaml` drives this process. Each declared type maps to a source file by naming convention and installs to a shard-qualified destination.

## Source Naming

Type definition source files live in the shard's `install/` directory and follow strict naming:

| Pattern | When |
|---------|------|
| `type-<sh>-<type>.md` | Root type (e.g. `type-proj-task.md`) |
| `type-<sh>-<type>_<subtype>.md` | Subtype (e.g. `type-f-note_concept.md`) |

- `<sh>` is the shard shorthand (lowercase letters, any length)
- `<type>` is the type name in lowercase, words joined by underscore
- `<subtype>` is the subtype name in lowercase after an underscore separator
- The underscore separating type from subtype is distinct from underscores within a multi-word name — the **first** underscore after the type stem marks the subtype boundary

**Derivation from `types:` declaration:**

| Declaration | Source file |
|-------------|------------|
| `Task` | `type-proj-task.md` |
| `Learning Report` | `type-lrn-learning_report.md` |
| `Note.Concept` | `type-f-note_concept.md` |

Multi-word type names become `lower_snake_case` in the source filename. For subtypes, the parent and child are separated by underscore in the filename since dash is the shorthand separator.

## Destination Naming

Installed type definitions land in `Mesh/Metadata/Types/` with the following naming pattern, conforming to [[MKD Title ID Protocol]]:

```
Mesh/Metadata/Types/(Type) <Name> (<Shard Display Name> Shard).md
```

For subtypes:

```
Mesh/Metadata/Types/(Type) <Name> . <Subname> (<Shard Display Name> Shard).md
```

- `(Type)` is a literal marker — all type definition files carry this type prefix
- `<Name>` is the human-readable type name, each word with a capital letter
- `<Shard Display Name>` is the name of the shard (the lock `name`), also when the shard has another alias in `flint.toml`
- ` Shard` is a literal suffix — part of the context qualifier
- Subtypes use `. <Subname>` (dot-space separator) per MKD untyped subfile convention

**Examples:**

| Declaration | Shard Name | Destination |
|-------------|------------|-------------|
| `Task` | Projects | `(Type) Task (Projects Shard).md` |
| `Notepad` | Notepad | `(Type) Notepad (Notepad Shard).md` |
| `Learning Report` | Learn | `(Type) Learning Report (Learn Shard).md` |
| `Note.Concept` | Flint | `(Type) Note . Concept (Flint Shard).md` |

The shard context qualifier prevents collisions — two shards with different names that declare the same type name produce distinct files. Two shards with one name (under two aliases) share the destination; the second install keeps the first file.

## shard.yaml Declaration

The `types:` field in `shard.yaml` is a string array. Each entry is either a simple type name or a `Parent.Child` subtype:

```yaml
types:
  - Task
  - Note.Concept
```

**Rules:**
- Each word with a capital letter; letters and numbers only (no spaces in single words, spaces between words)
- Format: `Type`, `Multi Word Type`, or `Type.Subtype`
- Pattern: `/^[A-Z][A-Za-z0-9]*(?: [A-Z][A-Za-z0-9]*)*(?:\.[A-Z][A-Za-z0-9]*(?: [A-Z][A-Za-z0-9]*)*)?$/`

**What `types:` does:**
- Installs the type definition file from `install/` to `Mesh/Metadata/Types/`
- Follows `once` semantics — does not overwrite if the destination already exists

**What `types:` does NOT do:**
- Does not create artifact storage folders under `Mesh/Types/`
- Does not create any directories

## Folders Decoupling

Artifact storage folders are created by the `folders:` field, not by `types:`. A shard that manages Tasks declares both:

```yaml
types:
  - Task                          # Installs type definition file

folders:
  - "Mesh/Types/Tasks"    # Creates artifact storage folder
  - Mesh/Archive/Tasks            # Creates archive folder
```

This decoupling means:
- A shard can declare a type without needing an artifact folder (e.g., a meta-type like `Shard`)
- A shard can create folders without declaring a type (e.g., shared archive directories)
- The relationship between types and folders is explicit rather than magical

## Type Definition File Structure

Type definition files are hand-authored markdown documents. They describe the type for agents and humans. Recommended structure:

```markdown
# <Type Name>

<Brief description of what this type represents.>

## Properties

| Property | Value |
|----------|-------|
| Tag | `#<sh>/<type>` |
| Naming | `(<Type>) NNN Name.md` |
| Location | `Mesh/Types/<Plural>/` |
| Archive | `Mesh/Archive/<Plural>/` |

## Lifecycle

\`\`\`
draft → active → archived
\`\`\`

## Templates

- [[tmp-<sh>-<type>-v<X.Y>]]
```

The `Tag` property in the Properties table is required by the health checker — it validates that type files declare their tag.

## Legacy Format

Older `shard.yaml` files (spec 0.1.0) used an object format:

```yaml
types:
  - name: Task
    source: type-proj-task.md
```

The manifest parser normalizes this to the string format automatically. The `source` field is ignored since the source filename is derived from the type declaration and shorthand by convention.

## Install Semantics

- **Install mode:** `once` — the install copies the file only if the destination does not exist. Users may change the file in the Mesh.
- **Template processing:** `{{uuid}}` and `{{date}}` placeholders are resolved at install time.
- **ID preservation:** If the destination file already exists (on reinstall with force), the existing `id` is preserved.
- **Readonly tag:** Every installed type file receives `#readonly` in its tags (also when the shard is a build of a local or a remote source).
- **Record:** The install lists each type file in `flint.json#shards[<id>].payloads[]` with `kind: type`, `sha256`, `mode`, and the Mesh `id`. Health reads the type files from this lock record, not from the `shard.yaml` of the build.
- **Rename:** `flint shard rename <alias> --name` (and the `moved` heal in a consumer Flint: `flint sync` for a rename that the source, the place, or the build shows, `flint shard install` for a rename that the registry reports) moves the type file to the new qualifier. The file keeps its `id`, and the `[[wikilinks]]` to it follow. See [[(Spec) Flint Shards . Rename]].
- **Reference:** A shard bound with `use = "reference"` (no build here) still installs its type files into the Mesh.
- **Uninstall:** When a shard is removed, its unchanged type definition files are deleted. A changed file stays.
